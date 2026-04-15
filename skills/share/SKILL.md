---
name: share
description: Publish, update, or remove HTML pages on sharable.link. Use when the user says "share this", "publish this", "make this shareable", "share link", "/share", "update shared page", "update link", "/update", "unshare", "/unshare", or "remove shared page". Supports optional password protection.
allowed-tools: Read, Bash, Glob
---

Publish an HTML file to sharable.link so it's accessible via a unique public URL, update an existing shared page, or remove a previously shared page.

---

## SHARE FLOW

Use this flow when the user wants to publish / share a page.

### Steps

1. **Find the HTML file to share:**
   - If the user provided a file path in `$ARGUMENTS`, use that file.
   - Otherwise, use Glob to find the most recently created `.html` file in the project (check common output locations: current directory, `out/`, `dist/`, `build/`, `output/`).
   - If multiple HTML files exist and none was specified, list them and ask which one to share.

2. **Check if this file was already shared in this conversation:**
   - If you have a slug and deleteKey from a previous share of this same file in this conversation, ask the user ONCE: "This file was shared earlier as sharable.link/{slug}. Do you want to **update that link** or **create a new one**?"
   - Remember their choice for the rest of the conversation. If they chose "update", use the UPDATE FLOW for all subsequent shares of the same file.
   - Only change behavior if the user explicitly says "create a new link", "new URL", or similar.
   - If the file was not previously shared, proceed with publishing.

3. **Read the file** using the Read tool. Verify it contains HTML content.

4. **Ask about password protection** (optional):
   - Ask the user: "Do you want to password-protect this link?"
   - If yes, ask for the password they want to use.

5. **Publish it** by running this curl command via Bash:

```bash
# Without password:
curl -sL -X POST https://sharable.vercel.app/api/publish \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg html "$(cat FILE_PATH)" '{html: $html}')"

# With password:
curl -sL -X POST https://sharable.vercel.app/api/publish \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg html "$(cat FILE_PATH)" --arg pw "PASSWORD" '{html: $html, password: $pw}')"
```

Replace `FILE_PATH` with the actual path to the HTML file, and `PASSWORD` with the user's chosen password.

6. **Return the result** — parse the JSON response and show the user the shareable URL and delete key. Format it clearly like:

> Published! Your sharable link: https://sharable.link/abc123
>
> Delete key: `a1b2c3d4...` — save this if you ever want to update or remove the page.

If password-protected, also add:

> This link is password-protected. Share the password separately with your recipients.

**If the request fails** (network error, timeout, or "blocked by allowlist"):

> The request to sharable.vercel.app was blocked. This is a Claude settings issue — not a bug in the skill.
>
> **Quick fix (30 seconds):**
> 1. Click your profile icon → **Settings**
> 2. Go to **Capabilities**
> 3. Under "Code execution and file creation", make sure **Allow network egress** is enabled
> 4. Change **Domain allowlist** to **All domains**
> 5. **Start a new chat** — settings changes don't apply to the current conversation
> 6. Try `/share` again in the new chat — it should work.
>
> Full guide with screenshots: https://sharable.link/blog/fix-claude-skill-network-access

---

## UPDATE FLOW

Use this flow when the user wants to update an existing shared page, or when the SHARE FLOW detects a re-share and the user chose "update".

### Steps

1. **Get the slug:**
   - If available from a previous share in this conversation, use it directly.
   - If the user provided a URL or slug in `$ARGUMENTS`, extract the slug (the part after `sharable.link/`).
   - Otherwise, ask: "Which page do you want to update? Provide the URL or slug."

2. **Get the delete key:**
   - If the delete key is available in the current conversation (e.g. from a recent share), use it directly.
   - Otherwise, ask: "What is the delete key for this page? (It was shown when you first shared it.)"

3. **Find and read the updated HTML file:**
   - Same logic as SHARE FLOW Step 1 — find the HTML file to use as the new content.
   - Read it using the Read tool.

4. **Ask about password changes** (optional):
   - If the user mentions changing or removing the password, handle it. Otherwise, the existing password stays as-is.

5. **Update it** by running this curl command via Bash:

```bash
# Update content only (keep existing password):
curl -sL -X POST https://sharable.vercel.app/api/update \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg slug "SLUG" --arg key "DELETE_KEY" --arg html "$(cat FILE_PATH)" '{slug: $slug, deleteKey: $key, html: $html}')"

# Update content and change/set password:
curl -sL -X POST https://sharable.vercel.app/api/update \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg slug "SLUG" --arg key "DELETE_KEY" --arg html "$(cat FILE_PATH)" --arg pw "PASSWORD" '{slug: $slug, deleteKey: $key, html: $html, password: $pw}')"

# Update content and remove password:
curl -sL -X POST https://sharable.vercel.app/api/update \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg slug "SLUG" --arg key "DELETE_KEY" --arg html "$(cat FILE_PATH)" '{slug: $slug, deleteKey: $key, html: $html, password: ""}')"
```

Replace `SLUG`, `DELETE_KEY`, `FILE_PATH`, and `PASSWORD` with actual values.

6. **Return the result** — parse the JSON response:

If successful:
> Updated! Your link is still: https://sharable.link/SLUG

If the password was changed, add:
> Password has been updated.

If the password was removed, add:
> Password protection has been removed. The page is now public.

**If the request fails** — show the same network error block as SHARE FLOW.

---

## UNSHARE FLOW

Use this flow when the user wants to remove / unshare / delete a previously shared page.

### Steps

1. **Get the slug:**
   - If the user provided a URL or slug in `$ARGUMENTS`, extract the slug (the part after `sharable.link/`).
   - Otherwise, ask: "Which page do you want to remove? Provide the URL or slug."

2. **Get the delete key:**
   - If the delete key is available in the current conversation (e.g. from a recent share), use it directly.
   - Otherwise, ask: "What is the delete key for this page? (It was shown when you first shared it.)"

3. **Delete it** by running this curl command via Bash:

```bash
curl -sL -X POST https://sharable.vercel.app/api/delete \
  -H "Content-Type: application/json" \
  -d "$(jq -n --arg slug "SLUG" --arg key "DELETE_KEY" '{slug: $slug, deleteKey: $key}')"
```

Replace `SLUG` with the page slug and `DELETE_KEY` with the user's delete key.

4. **Return the result** — parse the JSON response:

If successful:
> Done! The page sharable.link/SLUG has been removed.

If the key is wrong (403):
> The delete key is incorrect. Please check and try again.

If the page doesn't exist (404):
> That page was not found. It may have already been removed.
