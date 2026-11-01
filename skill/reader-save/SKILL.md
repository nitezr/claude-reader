---
name: reader-save
description: Save links, articles, tweets, videos and Instagram posts into my Reader artifact with a clean reading view, summary and key points. Use for "save this to my Reader", "add to reader", or "process my Reader queue".
---

# Reader: save links

My Reader is a claude.ai artifact built from this repo.

- Artifact: <PASTE YOUR ARTIFACT LINK OR SLUG HERE>
- Database (db capability) collections:
  - `docs/<id>`: one saved item (schema below)
  - `highlights/<id>`: `{docId, text, note, createdAt}`, created by the app when I highlight text
  - `queue/<id>`: `{url, addedAt, status}`, links I queued from the app's + button

## Doc schema
```json
{
  "title": "Article title",
  "url": "https://original-link",
  "source": "example.com",
  "author": "Name or @handle",
  "type": "article | tweet | video | instagram | pdf",
  "location": "inbox | later | archive",
  "savedAt": "2026-11-01T09:00:00Z",
  "published": "2026-10-28",
  "readMins": 6,
  "summary": "Two sentences max.",
  "keyPoints": ["3–5 short bullets"],
  "tags": ["lowercase", "one-word"],
  "image": "data: URI of a small cover image (optional)",
  "content": [{"k": "p|h2|h3|quote|li|code", "t": "text"}]
}
```

## When triggered
1. Collect links from the chat, and/or `db_list` the `queue` collection ("process my Reader queue").
2. For each link, fetch the page (WebFetch, or the browser for pages that need sign-in).
   - **Article:** keep the real paragraphs and headings in order and drop menus, ads and footers. Split into `content` blocks.
   - **Tweet or thread:** one `p` block per tweet.
   - **Video:** use the description or transcript and write notes as `content`. Put the main ideas in `keyPoints`.
   - **Instagram:** the caption, plus a list of what's shown.
3. Write `summary` (two sentences max) and 3–5 `keyPoints`. Guess `readMins` at about 230 words a minute. Pick 1–3 tags.
4. Optional cover: artifacts can't load outside images, so shrink the page's main image to about 400 px wide JPEG and store it as a `data:` URI in `image`. Skip it if it's over 60 KB.
5. Skip a link if a doc with the same `url` already exists (check with `db_query` or `db_list`).
6. `db_write` add to `docs` with `location: "inbox"` and `savedAt` = now. Mark processed `queue` items `status: "done"`.
7. Reply with one line per saved item: title, source, read time.

Only store content I have the right to keep for personal reading, and always keep the `url` to the original.
