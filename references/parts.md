# Writing parts

A story is an ordered list of 1 to 30 parts. Each part is one screen for the learner: story text (or a video), then grammar, vocabulary and a quiz. Parts are numbered by their position in the `parts` array, so array order is reading order.

## Plan the parts first

- Short story (A1–A2): 2–4 parts, 2–4 short paragraphs each.
- Longer story (B1+): 3–6 parts, each ending at a natural pause.
- Give every part a short `title` of its own ("Llegada", "El almuerzo"). Do not title them "Part 1".
- Keep new vocabulary and grammar inside the part where the words first appear.

## What a part contains

| Field | Required | Notes |
|---|---|---|
| `title` | yes | At most 120 characters. |
| `paragraphs` | text or video | One string. Separate paragraphs with a blank line (`\n\n`). Up to 20000 characters. |
| `imageKey`, `imageAlt`, `imageAfter` | no | Key from `create_media_upload`. Alt text is required with an image. `imageAfter` is the 1-based paragraph the image follows; use `1` when there is no image. |
| `audioKey` | no | Narration for the part's text. |
| `videoKey` | no | An uploaded video. |
| `youtube`, `youtubeStart`, `youtubeEnd`, `cues` | no | A YouTube video. Start, end and cues need a `youtube` link. End must come after start. |
| `grammar` | array, may be empty | Each point: `title`, `explanation`, and at least one `{ text, translation }` example. |
| `vocabulary` | array, may be empty | `{ word, meaning }`. |
| `quiz` | array, may be empty | `question`, 2–5 `options` (`{ text }`), `answer` as the zero-based index of the right option. |

A part needs paragraphs, a YouTube link or an uploaded video. Every field must be present; unused strings are `""` and unused lists are `[]`.

## Examples

### A story with three text parts

`payloads/create_story.multipart.json`. The shape of each part, abbreviated:

```json
{
  "title": "El almuerzo",
  "paragraphs": "Los hermanos comen en un restaurante pequeño.\n\nMarta pide una tortilla y Pablo pide pescado.",
  "imageKey": "", "imageAlt": "", "imageAfter": 1,
  "audioKey": "", "videoKey": "",
  "youtube": "", "youtubeStart": "", "youtubeEnd": "", "cues": "",
  "grammar": [],
  "vocabulary": [{ "word": "tortilla", "meaning": "Spanish omelette" }],
  "quiz": [
    { "question": "¿Qué pide Pablo?", "options": [{ "text": "Tortilla" }, { "text": "Pescado" }, { "text": "Ensalada" }], "answer": 1 }
  ]
}
```

### A part with an image after the second paragraph

Upload the image first (`create_media_upload`, PUT, keep the `key`), then:

```json
{
  "title": "En el parque",
  "paragraphs": "Pasean por el parque.\n\nEl lago es muy grande.\n\nCompran un helado.",
  "imageKey": "stories/media/3f2b8c1e-9a4d-4e6f-8b7a-1c2d3e4f5a6b.jpg",
  "imageAlt": "Dos hermanos caminan junto a un lago.",
  "imageAfter": 2,
  "audioKey": "", "videoKey": "",
  "youtube": "", "youtubeStart": "", "youtubeEnd": "", "cues": "",
  "grammar": [], "vocabulary": [], "quiz": []
}
```

### A video-only part

No text needed. See `payloads/create_story.youtube.json`. Cues are one per line in time order:

```
0:12 Hola, amigos. | Hello, friends.
0:20 Hoy toco para vosotros. | Today I play for you.
```

### A part with grammar

```json
"grammar": [
  {
    "title": "Presente de llegar",
    "explanation": "Regular -ar verb: llego, llegas, llega, llegamos, llegáis, llegan.",
    "examples": [{ "text": "Marta llega en tren.", "translation": "Marta arrives by train." }]
  }
]
```

## Changing the parts of an existing story

Parts have no ids and no part-level tools. Every change is `get_story`, edit the `parts` array, then `update_story` with the whole story. Anything left out is removed.

| Goal | Edit to the `parts` array | Payload |
|---|---|---|
| Add a part | Append (or insert) the new part object | `payloads/update_story.add_part.json` |
| Remove a part | Drop the object | `payloads/update_story.remove_part.json` |
| Reorder parts | Move the objects | `payloads/update_story.reorder_parts.json` |
| Edit one part | Change that object, keep the rest unchanged | |

Keep media keys, alt text and quiz answers on the parts you did not touch exactly as `get_story` returned them. If you insert a part before an existing one, the later parts shift down by one number.

## Checks before saving

- 1–30 parts, each with a title.
- Each part has paragraphs, a YouTube link or a video.
- Every `answer` is a valid option index (`0` to options − 1).
- Every image has alt text.
- Cues are in time order and `youtubeEnd` is later than `youtubeStart`.
