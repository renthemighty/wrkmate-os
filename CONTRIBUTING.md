# Contributing to Liquid OS

Liquid OS is early. Most of the project is still spec and design, not code. That makes this a good time to shape it, and a good time to keep changes small.

## Before you write code

Open an issue first. Describe the problem or the piece of the spec you want to build. This applies to every layer: resolver, activation engine, Android shell, iOS shell. Get a thumbs up before you spend real time on it. Ideas that skip the spec are much harder to merge later.

## Pull requests

- Keep them small and focused on one change.
- Reference the issue it closes or advances.
- Explain what you tested and how.
- Match the style of the surrounding code. If a file uses tabs, use tabs. If a module has a naming convention, follow it. Do not introduce a second style in a file that already has one.

## License

All contributions are made under GPL-3.0, the same license as the rest of the project. By opening a pull request you agree your contribution is licensed under GPL-3.0.

## Scope

If you are proposing a new subsystem (a new entity source for the resolver, a new tool type for the activation engine, a platform beyond Android and iOS), put it in an issue and explain the use case before building it. Big additions need a spec update before a code change.
