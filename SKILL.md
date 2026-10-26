---
name: brutalingo-mcp
description: Create, edit, publish and delete Brutalingo Spanish-learning stories through the Brutalingo MCP server. Use when the user asks to write, save, update, publish, unpublish or delete a Brutalingo story, upload story images, audio or video, or mentions the brutalingo MCP tools (list_stories, get_story, create_story, update_story, set_story_status, delete_story, create_media_upload).
---

# Brutalingo MCP

Manage stories in the Brutalingo admin from an LLM client. The server is the app's own `/api/mcp` endpoint (stateless Streamable HTTP, OAuth 2.1 with PKCE). Only admins listed in `ADMIN_EMAILS` can connect.

## Connect

The user does this once. The admin page `/admin/mcp` shows the exact URL and a copy button.

- Claude.ai / Claude Desktop: Settings → Connectors → Add custom connector → paste `https://<app-host>/api/mcp`.
- Claude Code: `claude mcp add --transport http brutalingo https://<app-host>/api/mcp`, then `/mcp` to sign in.
- Sign in as an admin and press Approve on the consent page. Access tokens last 1 hour and refresh on their own for 30 days.

If the tools are missing, the connector is not approved yet or the signed-in account is not an admin. Tell the user; do not guess a URL.

## Workflow

1. Read the `brutalingo://story-format` resource. It holds the fields, limits and a full example. Do not write a story from memory of the format.
2. Call `list_stories` so the slug is unique and the `category` matches an existing one when it fits.
3. Write the story in the target language (`es-ES` by default), graded to the requested CEFR level.
4. Upload media first if any is needed (see below), then call `create_story` with `status: "DRAFT"`.
5. Report the slug and what was saved. Publish only when the user asks.

The `write_story` prompt (topic, level, lang, parts) bundles steps 1–4 for a fresh story.

## Tools

| Tool | Use |
|---|---|
| `list_stories` | Slug, title, level, category, lang, status, part count. Optional `status` filter. Deleted stories are hidden. |
| `get_story` | One story in the same shape `create_story` takes, plus its status. |
| `create_story` | New story. `status` defaults to `DRAFT`. Fails on a duplicate slug. |
| `update_story` | **Full replace** of a story's content. Slug cannot change. Omitting `status` keeps the current one. |
| `set_story_status` | `PUBLISHED` makes it visible to every learner, `DRAFT` hides it. |
| `delete_story` | Soft delete. An admin restores it from the admin list; there is no restore tool. |
| `create_media_upload` | Returns a 15-minute signed PUT URL for an image, audio or video file. |

Ready-to-adapt arguments for every tool, with response shapes, are in `references/payloads/` (see its `README.md`). Start from `create_story.json` for a text story and `create_story.youtube.json` for a video part.

How to plan, write, add, remove and reorder parts, with examples, is in `references/parts.md`.

## Rules that matter

- **Updating:** call `get_story` first, change what is needed, send the whole story back. Parts left out are removed.
- **Never publish or delete on your own.** Do it only on an explicit request naming the story. Say which slug you changed.
- **Every field is required.** Unused strings are `""`, unused lists are `[]`; none may be left out.
- Slug: lowercase letters, numbers and single dashes, at most 80 characters. Fixed after creation.
- `paragraphs` is one string, paragraphs separated by a blank line. A part needs text, a YouTube video or an uploaded video.
- Quiz `answer` is the zero-based index of the correct option; each question has 2–5 options. A part has grammar points (with at least one example), vocabulary and a quiz.
- An image needs `imageAlt`. `imageAfter` is the paragraph the image follows.
- Validation errors come back as a list of messages. Fix them and retry rather than loosening the content.

## Media

1. `create_media_upload` with `kind` (`image`, `audio`, `video`) and the file's `content_type`.
2. PUT the raw bytes to the returned `url` with the same `Content-Type`, within `expires_in_seconds` and under `max_bytes`. The JSON response contains `key`.
3. Put that `key` in the part's `imageKey`, `audioKey` or `videoKey`.

Unsupported types are refused with the allowed list. A key that does not exist in storage fails validation on save.

## Errors

- `401` / sign-in prompt: the token expired or the account is not an admin. Reconnect from the client.
- `429`: more than 300 requests a minute. Wait and retry.
- `No story with slug "…"`: check `list_stories`; deleted stories are not visible.
