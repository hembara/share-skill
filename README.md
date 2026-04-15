# /share — Turn Claude outputs into shareable links

A Claude skill that publishes HTML files to [sharable.link](https://sharable.link) and gives you a clean public URL. Also supports unpublishing pages with `/unshare`.

## Install

### Via skills CLI

```bash
npx skills add hembara/share-skill
```

### Manual install

1. Download [`SKILL.md`](skills/share/SKILL.md)
2. In Claude, go to **Customize > Skills > + > Create skill > Upload a skill**
3. Upload the file

Full walkthrough with screenshots: [sharable.link/guide](https://sharable.link/guide)

## Prerequisites

Make sure your Claude settings allow network access:

- **Settings > Capabilities > Allow network egress** — enabled
- **Domain allowlist** — set to **All domains**
- **Start a new chat** after changing settings

Having issues? See the [network access fix guide](https://sharable.link/blog/fix-claude-skill-network-access).

## Usage

### Share a page

Ask Claude to build something (a dashboard, report, landing page), then say:

```
/share
```

Claude publishes the HTML and returns:
- A public URL anyone can open
- A delete key for removing the page later

### Unshare a page

```
/unshare
```

Provide the page URL and your delete key. The page is removed immediately.

### Password protection

Say "share this with a password" and Claude will ask you to set one. Recipients need the password to view the page.

## What it does

- Publishes any HTML file to a unique URL on [sharable.link](https://sharable.link)
- No account, no deployment, no server needed
- Supports password-protected links
- Works with Claude Cowork, Claude Code, and any agent that supports skills

## Links

- [sharable.link](https://sharable.link) — homepage
- [Install guide](https://sharable.link/guide) — step-by-step with screenshots
- [Blog](https://sharable.link/blog) — guides and tips
- [Fix network errors](https://sharable.link/blog/fix-claude-skill-network-access) — if the skill can't reach the server

## License

MIT
