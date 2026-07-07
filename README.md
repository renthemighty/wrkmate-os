# WrkMate OS

An open source phone interface, built on Liquid Technology, that finishes tasks while you are still speaking.

## What it is

WrkMate OS replaces the wake-word, open-app, fill-in-fields, confirm flow that Siri and Google Assistant use. Instead of waiting for a full command and then acting, it streams speech into an on-device resolver that starts working the moment it recognizes a keyword. Say "send a text to Sarah, I'll be there in 10 minutes" and the resolver identifies the contact and starts drafting the message while you are still talking. By the time you stop speaking, the action is already done.

Heavier requests are handed to the activation engine, a server-side build system that assembles one-off tools on demand. Ask "how big is my yard" and the engine fetches a satellite image, works out scale, outlines the lot, and returns a square footage number. Nobody wrote a yard-measuring app ahead of time. The engine built the tool for that request.

Think of it the way Windows sat on DOS. The existing operating system keeps running underneath and keeps doing the low-level work it is good at. WrkMate OS runs on top of it, within each platform's constraints, and becomes the layer you actually live in. It does not pretend to replace what the platform will not let it replace.

## Liquid Technology

Liquid Technology is the set of skills at the center of WrkMate OS: single-purpose tools written on the fly, in real time, to achieve the goal the user just spoke, and then removed when the task completes. Nothing stays resident. That keeps the device light: storage stays free, transfer speeds stay high, battery is not spent idling on background processes nobody asked to run.

User content is untouched by this cycle. Photos, messages, and files are stored normally and sent as packages, the same as on any phone. Liquid Technology only governs the tools, not the data.

WrkMate OS is voice-first with a deliberately simple interface. Users who want icons can keep icons. Past that first screen there are no apps as such, only Liquid Technology assembling and discarding tools per task.

## How it works

WrkMate OS is three layers.

**Resolver.** Runs on the device. Streams the microphone input against a local keyword and entity library (contacts, calendar, recent apps) and starts matching intent before the sentence finishes. It does not wait for silence to begin working.

**Activation engine.** Runs on a server, self-hostable or hostable on EasWrk. Receives partial intent frames from the resolver as they arrive, assembles whatever tool or response the request needs, and streams the result back in parallel with the rest of the sentence being spoken.

**Shell.** The interface the user actually sees and talks to. On Android, with an unlocked bootloader, it replaces the stock launcher and interface outright. On iOS it installs as the first app on a clean device and becomes the front door: it drives the phone's existing subsystems (contacts, messages, camera, maps) directly instead of opening full apps for each one. Fewer processes running, less battery burned per task.

The resolver and the engine run on different scales of model on purpose. The resolver carries a tiny gate model sized to the device at install, matching keywords and entities only, never generating language. The engine sits behind a model-agnostic routing layer that can call any large model: Claude, GPT, Grok, Gemini, self-hosted Llama, and the Chinese providers DeepSeek, Qwen, Kimi, and GLM, so no market is left out and a region blocked from one provider still gets routed to another.

## Platform story

**Android.** On devices with an unlocked bootloader, WrkMate OS takes over as the primary interface. No app grid underneath it to fall back to.

**iOS.** Apple does not allow a full system replacement, so WrkMate OS installs as the first app on the device and acts as the front door for everything voice can reach. It calls into Messages, Contacts, Camera, and Maps directly rather than spawning each app in turn.

## Project status

Early scaffold. This repository is documentation, specs, and project layout only. Nothing here is a shipping product yet. The resolver, the activation engine, and both shells are all unbuilt. See `docs/roadmap.md` for the build order.

## Repo layout

| Path | Contents |
|------|----------|
| `docs/vision.md` | Why this project exists |
| `docs/architecture.md` | The three layers and how a request flows through them |
| `docs/roadmap.md` | Build phases, no dates |
| `spec/intent-frame.md` | JSON format the resolver streams to the engine |
| `spec/keyword-library.md` | On-device keyword and entity matching design |
| `spec/model-routing.md` | Provider routing config: model-agnostic engine, device resolver parameter budget, data boundary (draft) |
| `resolver/` | On-device streaming resolver (prototype not yet started) |
| `engine/` | Activation engine service (not yet started) |
| `shell/android/` | Android launcher replacement (not yet started) |
| `shell/ios/` | iOS front-door app (not yet started) |

## License

GPL-3.0. See `LICENSE`.

## Credit

Developed by Simon Painter at EasWrk (easwrk.com).
