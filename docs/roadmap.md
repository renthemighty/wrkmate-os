# Roadmap

No dates. Phases represent build order, not a schedule.

## Phase 0: Scaffold and spec

Repository structure, vision and architecture docs, intent frame spec, keyword library spec. This phase produces no running code. Its job is to make the next phases buildable without guesswork. This is where the project is right now.

## Phase 1: Resolver prototype

A standalone on-device resolver that takes streaming speech input, matches against a small keyword and contact library, and emits intent frames matching the spec in `spec/intent-frame.md`. No shell, no engine. Success looks like: correct intent frames emitted at the right time, with partial frames arriving before the sentence ends.

## Phase 2: Activation engine service

A server that accepts intent frames over the wire, handles a small set of built-in actions (compose message, set timer, place call), and can be self-hosted. This phase proves the frame protocol and the parallel-processing model, without yet doing on-demand tool assembly.

## Phase 3: Android shell

A launcher replacement for Android devices with an unlocked bootloader. Wires the resolver and the engine into a real interface: microphone always ready, contact and calendar access, confirmation UI. This is the first phase where a user could actually run WrkMate OS as their daily interface, on supported hardware.

## Phase 4: iOS front door app

An iOS app that installs as the first app on a clean device and drives Messages, Contacts, Camera, and Maps directly through supported system APIs. Built to the constraints Apple actually allows, not a workaround of them.

## Phase 5: Tool assembly on demand

The activation engine gains the ability to build new one-off tools for requests it has not seen a fixed action for, starting with the yard measurement example: fetch imagery, compute scale, outline a boundary, return a result. This is the phase that turns the engine from a fixed action dispatcher into a real build system.
