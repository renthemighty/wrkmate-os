# Intent Frame Spec

An intent frame is the message the resolver streams to the activation engine. The resolver emits a new frame each time its understanding of the request changes meaningfully, not just once at the end. A request typically produces several frames: an early partial frame with low confidence, and a final frame once the sentence completes.

## Fields

| Field | Type | Description |
|-------|------|-------------|
| `sequence` | integer | Frame number within this request, starting at 1. Increases with each new frame the resolver emits for the same utterance. |
| `partial_transcript` | string | The speech-to-text transcript so far, up to this frame. |
| `matched_keywords` | array of strings | Keywords from the keyword library matched so far (for example `["text"]` or `["text", "send"]`). |
| `candidate_action` | string | The action the resolver believes is being requested (for example `compose_message`, `set_timer`, `place_call`). Can change between frames if the resolver revises its guess. |
| `confidence` | number | 0 to 1. The resolver's confidence in `candidate_action` as it currently stands. |
| `entities` | object | Named entities resolved so far. Keys depend on the action (see below). |
| `final` | boolean | True only on the last frame for this utterance, once the resolver considers the sentence complete. |

## Entities object

Shape depends on `candidate_action`. For `compose_message`:

| Key | Type | Description |
|-----|------|-------------|
| `contact` | string or null | Matched contact name, or null if not yet resolved. |
| `contact_id` | string or null | Stable identifier for the matched contact once resolved. |
| `message_body` | string or null | The message text, or null if not yet spoken. |

## Example: partial frame

```json
{
  "sequence": 1,
  "partial_transcript": "send a text to Sarah",
  "matched_keywords": ["text"],
  "candidate_action": "compose_message",
  "confidence": 0.7,
  "entities": {
    "contact": "Sarah",
    "contact_id": "contact_8842",
    "message_body": null
  },
  "final": false
}
```

## Example: final frame

```json
{
  "sequence": 2,
  "partial_transcript": "send a text to Sarah, I'll be there in 10 minutes",
  "matched_keywords": ["text"],
  "candidate_action": "compose_message",
  "confidence": 0.98,
  "entities": {
    "contact": "Sarah",
    "contact_id": "contact_8842",
    "message_body": "I'll be there in 10 minutes"
  },
  "final": true
}
```

## Notes

The engine should treat every non-final frame as a hint to start preparing, not as a command to execute. Only a `final: true` frame with `candidate_action` and its required entities present should trigger the actual action. The engine may use partial frames to warm caches, pre-load contact data, or reserve a compose session, but it should not send a message, place a call, or take any other irreversible action until it receives a final frame.
