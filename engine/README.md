# Activation Engine

This directory will hold the activation engine service: the server-side build system that receives intent frames from the resolver, assembles the tools or responses a request needs, and returns completed actions while the user is still speaking. Self-hostable, and also hostable on EasWrk.

Nothing is built here yet. This is a phase 2 item on the roadmap, with on-demand tool assembly (the yard measurement example in `docs/vision.md`) landing later in phase 5. See `docs/architecture.md` for how the engine fits between the resolver and the shell, and `spec/intent-frame.md` for the message format it consumes.
