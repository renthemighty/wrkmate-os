# Keyword and Entity Library Spec

The keyword library is what runs on the device to turn a stream of words into an intent frame. It has two halves: action words and entity sources.

## Action words

A small set of verbs and short phrases the resolver watches for as speech comes in. Each one maps to a candidate action:

| Keyword | Candidate action |
|---------|-------------------|
| text, message, send a text | `compose_message` |
| call, dial, phone | `place_call` |
| timer, set a timer, remind me in | `set_timer` |
| measure, how big is, how big are | `measure_area` |

This table is a starting point, not a fixed list. The library is designed to grow. New action words map to new candidate actions the activation engine knows how to handle, or in phase 5, actions it can assemble on the fly.

The resolver does not require the action word to appear first. It watches the whole stream and matches whenever a keyword appears, then backfills the candidate action for the frame.

## Entity sources

Entities are the specific things a request refers to: a contact, a calendar event, a duration, a location. The resolver draws these from sources already available on the device:

- **Contacts**: names and identifiers from the phone's address book. Matched against spoken names as they arrive, including partial matches ("Sarah" can match before a last name is spoken, if only one Sarah exists).
- **Calendar**: event titles and times, for actions like scheduling or checking availability.
- **Recent locations**: places from Maps history, for actions that reference "home," "work," or a recently visited address.

Entity sources are read locally. The resolver does not need network access to identify who "Sarah" is; it needs network access (via the activation engine) to act on that identification, like actually sending a message through the messaging subsystem.

## How partial matches trigger early resolution

The resolver does not wait for a complete sentence to start narrowing possibilities. As soon as an action keyword is heard, it opens a candidate action and starts watching for the entities that action needs. As soon as an entity source finds a match, even a partial one (first name only, ambiguous until enough context arrives), the resolver includes it in the next intent frame with an honest confidence score. Confidence rises as more of the sentence disambiguates the request. This is what lets the activation engine start warming up work before the sentence is finished: it is reacting to a stream of increasingly confident partial frames, not a single command issued at the end.

## Growing the library

The keyword library is meant to be extended, not treated as fixed. Adding a new action word and mapping it to a new candidate action is the expected way to add a new capability to the resolver side. See `docs/roadmap.md` phase 5 for how the activation engine handles actions that do not have a pre-built handler yet.
