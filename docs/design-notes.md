# Design Notes

This file is the working record for the build. Entries are dated, newest at top. As specs firm up, decisions here get folded into the formal spec and architecture documents.

## 2026-07-07

### Naming
The product is WrkMate OS. The technique underneath the resolver and activation engine is called Liquid Technology. The working name Liquid OS is retired.

### Platform phrasing rule
Public and internal copy must never claim WrkMate OS replaces iOS. On Android, the shell takes over the interface only on devices with an unlocked bootloader; describe this the way DOS is described relative to Windows, an underlying layer the shell sits on top of and drives, not a wholesale replacement claim. A full Android replacement claim is only accurate on unlocked bootloaders. On iOS, WrkMate OS installs as the first app on a clean device; it is never described as replacing iOS.

### Model split
Two different models do two different jobs.
- Device gate model: runs the resolver, parameter budget from 20,000 to 160,000 params, chosen automatically by an install-time benchmark of the device it lands on. Slower or older phones get the smaller end of the range; current flagships get the larger end. The role stays identical at any size in the range, only matching speed and width change.
- Activation engine model: runs on a server, self-hosted or hosted on EasWrk, and is model agnostic. A routing layer sits in front of it and can key into Claude, GPT, Grok, Gemini, self-hosted Llama, and the Chinese providers DeepSeek, Qwen, Kimi, and GLM. Region pinning keeps requests inside a given jurisdiction's approved providers. Only structured intent frames leave the device toward this engine, never raw audio or full transcripts.

### Phrase command matching (resolver mechanics)
This is the mechanism behind Liquid Technology on the device side.

1. Each word is pattern matched as it passes, target latency in the hundredths of a second per word. Matching pulls from an on-device library of phrase commands to work out which function will be required.
2. Possibility pruning is the core trick. A word opens the set of things it could imply, and the next word cuts that set down. Canonical example: the word "send" opens the short list of sendable things, a text, an email, not much else. "Send a text" then drills straight down to one function. The action item is found through cheap elimination against a known list, not through deep language understanding.
3. As matches establish, the system spawns multiple processes to handle multiple statements at once, in real time, while the user is still talking. Stacked or compound requests resolve side by side rather than one after another.
4. This pruning is why the tiny parameter budget works. A 20,000 parameter model does not need to understand language broadly. It only needs to keep cutting the list of possibilities down. That is how a working amount of functionality fits into very few parameters.
5. A model at that scale, a Llama-class model cut down to roughly 20,000 parameters, runs entirely on the user's device. This resolution path needs no external source and no network connection. The activation engine only gets involved once the resolved function needs server-side work.
