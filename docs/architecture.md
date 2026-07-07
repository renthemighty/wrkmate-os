# Architecture

WrkMate OS has three layers. Each one has a single job.

## Shell

The interface the user sees and talks to.

On Android, on a device with an unlocked bootloader, the shell replaces the stock launcher and interface outright. There is no app grid underneath waiting to be revealed. The shell is the whole interface.

On iOS, Apple does not permit a full system takeover, so the shell installs as the first app on a clean device and acts as the front door. It listens continuously while active and drives the phone's existing subsystems directly: Contacts, Messages, Camera, Maps. It does not open the Messages app and hand off to it. It calls into the same functionality Messages uses and shows the result inline. Fewer processes spawned per task, less battery spent per task.

Past the first screen, the shell shows no app grid. What looks like a static interface is Liquid Technology underneath: the shell surfaces whatever tool the activation engine just assembled for the current request, and nothing else persists once the request is done.

## Resolver

Runs on the device, close to the microphone. Streams incoming speech against a local keyword and entity library: action words like "text," "call," "timer," "measure," plus entity sources like the contact list and calendar. The resolver does not wait for the user to finish speaking before it starts. It begins matching against the keyword library on the first recognized action word and starts narrowing candidate entities (which contact named Sarah, which calendar event) as soon as those words arrive.

The resolver is intentionally light. It does not do the heavy lifting itself. Its job is to identify what the user wants fast enough that the heavy lifting can start in parallel with the rest of the sentence, and to package what it knows so far into an intent frame it can send onward.

## Activation engine

Runs on a server: self-hosted, or hosted on EasWrk. Receives intent frames from the resolver as they arrive, not just at the end. If the request needs a tool that does not exist yet ("how big is my yard"), the engine assembles it on demand: fetches the satellite image, works out scale from known reference points, outlines the lot boundary, computes area, and returns the number. If the request is simple (send this text), the engine confirms the resolver's candidate action and returns quickly.

This assemble, use, discard cycle is Liquid Technology. The engine writes a single-purpose tool for the request just spoken, runs it, and removes it once the result is returned. No tool sits resident waiting for the next request that might never come.

The engine is built to be self-hostable and is not tied to running on any single infrastructure provider. EasWrk hosting is an option, not a requirement.

## Example flow: "send a text to Sarah, I'll be there in 10 minutes"

```
t=0.0s   user starts speaking: "send a text to"
t=0.3s   resolver matches action keyword "text"
t=0.3s   resolver opens candidate action: compose_message
t=0.6s   user says: "Sarah"
t=0.7s   resolver matches contact "Sarah" against contact library
t=0.7s   resolver emits intent frame #1 (partial): action=compose_message,
         contact=Sarah, body=null, confidence=0.7
t=0.7s   activation engine receives frame #1, opens a compose session,
         starts pre-loading Sarah's contact details in parallel
t=1.0s   user says: "I'll be there in 10 minutes"
t=2.4s   resolver finishes transcribing the body text
t=2.4s   resolver emits intent frame #2 (final): action=compose_message,
         contact=Sarah, body="I'll be there in 10 minutes",
         confidence=0.98, final=true
t=2.5s   activation engine finalizes the action: sends the message
t=2.5s   shell shows confirmation, message already sent
```

The design goal is the gap between t=2.4s (user stops talking) and t=2.5s (action complete). That gap should be small enough that the user experiences the phone as having understood them, not as having processed a command afterward. Most of the actual work, contact resolution, session setup, happens before the user finishes the sentence, so there is very little left to do once they stop.

## Model split

The resolver and the activation engine are not two sizes of the same model. They are different kinds of model doing different jobs.

The resolver's internal model is small on purpose. It is not a conversational model and it never generates language. Its only job is to match action keywords and entities from partial speech in microseconds, offline, at near-zero battery cost. Its parameter budget is not fixed. It is a moving target sized at install time to the device it runs on, roughly 20,000 parameters on slow or old phones, up to about 160,000 on current flagships. The role is identical at every size. A bigger budget on a faster phone means faster or wider matching against the keyword and entity library, not a smarter or more conversational resolver.

Language understanding and tool assembly happen entirely in the activation engine, and the engine is model agnostic. A provider routing layer sits in front of the engine and keys into any large model: Claude, GPT, Grok, Gemini, self-hosted Llama, and the Chinese providers DeepSeek, Qwen, Kimi, and GLM. Two reasons drive this. First, no market gets discounted by tying the engine to one provider. Second, when a region is blocked from one provider, the engine routes the same request to another without the resolver or the shell knowing the difference. Provider choice is configuration, not code. Swapping a provider means changing a routing entry, not rebuilding the engine. See `spec/model-routing.md` for the routing config format.

Region pinning and secure channels protect what leaves the device. Raw audio and full transcripts never leave the phone. Only structured intent frames are sent onward, and those frames can be pinned to providers operating in a given jurisdiction. Each provider connection runs over its own encrypted channel with its own keys, so a compromise of one provider connection does not expose another.
