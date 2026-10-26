# Example payloads

Each file is the `arguments` object for the tool of the same name. The story payloads pass the app's `storySchema`.

| File | Tool | Notes |
|---|---|---|
| `list_stories.json` | `list_stories` | `status` is optional. Leave the object empty `{}` for every non-deleted story. |
| `get_story.json` | `get_story` | |
| `create_story.json` | `create_story` | Text part with image, narration audio, grammar, vocabulary and quiz. The two media keys are placeholders: use keys returned by `create_media_upload`. |
| `create_story.youtube.json` | `create_story` | Video part: no text, YouTube link with start and end, subtitle cues. |
| `create_story.multipart.json` | `create_story` | Three text parts, each with vocabulary and a quiz. |
| `update_story.add_part.json` | `update_story` | Whole story with a fourth part appended. |
| `update_story.remove_part.json` | `update_story` | Whole story with the middle part dropped. |
| `update_story.reorder_parts.json` | `update_story` | Whole story with parts 2 and 3 swapped. |
| `update_story.json` | `update_story` | Full replace. Send the whole story from `get_story`, with changes. `status` omitted keeps the current one. |
| `set_story_status.json` | `set_story_status` | `PUBLISHED` or `DRAFT`. |
| `delete_story.json` | `delete_story` | Soft delete. |
| `create_media_upload.json` | `create_media_upload` | `kind` is `image`, `audio` or `video`. |

## Responses

Results are JSON text in the first content block.

`list_stories`
```json
[
  { "slug": "el-mercado-de-sevilla", "title": "El mercado de Sevilla", "level": "A2", "category": "Daily life", "lang": "es-ES", "status": "DRAFT", "updatedAt": "2026-10-26T09:00:00.000Z", "parts": 1 }
]
```

`get_story`: `{ "status": "DRAFT", "story": { ...same shape as create_story.story } }`

`create_story`, `update_story`, `set_story_status`: `{ "slug": "el-mercado-de-sevilla", "status": "DRAFT" }`

`delete_story`: `{ "slug": "el-mercado-de-sevilla", "deleted": true }`

`create_media_upload`
```json
{
  "method": "PUT",
  "url": "https://<app-host>/api/mcp/upload?token=<signed-token>",
  "headers": { "Content-Type": "image/png" },
  "max_bytes": 10485760,
  "expires_in_seconds": 900
}
```
`max_bytes` varies by kind: 10 MB image, 50 MB audio, 200 MB video. After the PUT, the upload endpoint answers `{ "key": "stories/media/<uuid>.png" }`.

## Errors

Failures set `isError: true` and the text lists the message, then one validation detail per line:

```
No story with slug "el-mercado-de-sevilla".
```
```
Unsupported image type "image/bmp". Allowed: image/png, image/jpeg, ...
```
