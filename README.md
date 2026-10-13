# Obsidian Wallabag Plugin (maintained fork)

Sync your [Wallabag](https://wallabag.org) articles into [Obsidian](https://obsidian.md) as notes, as PDFs, or as annotation-only notes.

This is a maintained fork of [huseyz/obsidian-wallabag](https://github.com/huseyz/obsidian-wallabag) by Hüseyin Zengin. It is based on upstream `0.3.2` and adds new features on top.

## What's new in this fork

- **Sync content mode.** Choose between *Full article* (the upstream behavior) and *Annotations only*. In annotations-only mode, a note contains only your Wallabag highlights and the notes you attached to them.
- **`{{annotations}}` template variable.** Renders all highlights and their notes as Markdown blockquotes. You can use it in any custom template.
- **Built-in annotations template** with `wallabag_id` in the frontmatter, so the *Delete note and remove it from synced articles cache* command works on these notes out of the box.
- Articles without annotations are skipped in annotations-only mode and are **not** marked as synced. They are picked up automatically on a later sync, once you highlight something in Wallabag.

## Features

- One-way sync from Wallabag to Obsidian. Each article is created once and is not overwritten later.
- Filter by Wallabag tag, and by unread and/or archived status.
- Separate target folders for unread and archived articles.
- Built-in templates or your own template file, with 17 template variables.
- Optional HTML to Markdown conversion of the article content.
- Optional PDF export of articles (via Wallabag's export API), with or without an accompanying note.
- Optional archiving of articles in Wallabag after they are synced.
- Sync on startup, from the command palette, or from the ribbon icon.

## Installation

This fork has no GitHub releases yet, so you build it from source:

```bash
git clone https://github.com/chrisOTM/obsidian-wallabag-2026.git
cd obsidian-wallabag-2026
npm install
npm run build
```

Then:

1. Create the folder `<VAULT>/.obsidian/plugins/wallabag/`.
2. Copy `main.js`, `manifest.json` and `styles.css` into that folder. `styles.css` is required because it hides the login fields once you are authenticated.
3. Reload Obsidian, turn off Restricted (Safe) Mode, and enable **Wallabag** under *Settings → Community plugins*.

Requires Obsidian 1.1.0 or newer.

> [!NOTE]
> This fork uses the same plugin ID (`wallabag`) as the upstream plugin in the community store, so it **replaces** the upstream plugin when you install it. If upstream ever publishes a version newer than `0.3.2`, Obsidian's plugin updater may overwrite this fork with that version.

## Authentication

1. In the Wallabag web UI, open **API clients management** and click **Create a new client**. Note the *Client ID* and *Client secret*.
2. In the plugin settings, set **Server URL** to your instance, for example `https://app.wallabag.it`. Do not include a trailing slash.
3. Under **Authentication**, enter the Client ID, Client secret, your Wallabag username and your password, then click **Authenticate**.

Your username and password are only used to request an OAuth token and are not stored. The token is refreshed automatically when it expires. If the refresh fails, the plugin logs you out and asks you to authenticate again.

## Usage

### Commands

| Command | What it does |
| :--- | :--- |
| **Sync Wallabag Articles** | Fetches articles that match your settings and creates notes (or PDFs) for the ones that have not been synced yet. The ribbon icon runs the same command. |
| **Clear synced articles cache** | Forgets which articles have been synced. The next sync recreates every matching article, except where a note with the same filename already exists. |
| **Delete note and remove it from synced articles cache** | Moves the active note to the trash and removes its article from the cache, so the next sync fetches it again with its current tags and annotations. The note's frontmatter must contain `wallabag_id` (see [Templating](#templating)). |

The plugin keeps track of synced article IDs. If you delete a note manually, the article is **not** synced again until you clear the cache. If a note with the target filename already exists, it is skipped and never overwritten.

### Settings

| Setting | Default | Description |
| :--- | :--- | :--- |
| Server URL | — | Base URL of your Wallabag instance, without a trailing slash. |
| Tag to sync | — | Only sync articles with this Wallabag tag. Leave it empty to sync all articles. |
| Wallabag article notes folder location | vault root | Folder for synced notes. |
| Article note template file | — | Vault path of a custom template, **without** the `.md` extension, for example `Templates/Wallabag`. Leave it empty to use the built-in templates. |
| Sync on startup | off | Run a sync when Obsidian starts. |
| Sync unread articles | on | Include unread articles. |
| Wallabag unread article notes folder location | — | Optional folder for unread articles. Falls back to the article notes folder. |
| Sync archived articles | off | Include archived articles. At least one of *unread* and *archived* must be enabled. |
| Wallabag archived article notes folder location | — | Optional folder for archived articles. Falls back to the article notes folder. |
| Archive article after sync | off | Archive each article in Wallabag after its note is created. The note is filed by the article's status at sync time, so it goes to the unread folder. |
| Convert HTML Content extracted by Wallabag to Markdown | off | Convert `{{content}}` from HTML to Markdown. |
| Sync content mode | Full article | *Full article* or *Annotations only*. In annotations-only mode, articles without annotations are skipped until they have some. |
| Add article ID in the title | off | Append `-<id>` to the note filename. This avoids collisions between articles that have the same title. |
| Tag format | CSV | `{{tags}}` as `tag1, tag2` (CSV) or `#tag1 #tag2` (Hashtags). |
| Export as PDF | off | Download each article as a PDF instead of creating a Markdown note. |
| PDF Folder | vault root | Folder for exported PDFs. Shown only when PDF export is enabled. |
| Create note | off | When PDF export is enabled, also create a note for each PDF in the article notes folder. |

Note filenames are built from the article title. The characters `\ , # % & { } / * < > $ " @ . ? : |` are replaced with spaces.

## Templating

The plugin ships three built-in templates. Which one is used depends on your settings:

| Situation | Built-in template |
| :--- | :--- |
| Full article mode | Tags in frontmatter, title, links to the original and to Wallabag, then `{{content}}` |
| Export as PDF + Create note | Same header as above, then a wikilink to the PDF (`{{pdf_link}}`) |
| Annotations-only mode | `tags` and `wallabag_id` in frontmatter, title, original and Wallabag links, then `{{annotations}}` |

![](screenshots/ss1.png)

If you set a custom template file, it is used in every mode. These variables are replaced:

| Variable | Description |
| :--- | :--- |
| `{{id}}` | Wallabag ID of the article. Put it in the frontmatter as `wallabag_id: {{id}}` to use the *Delete note and remove it from synced articles cache* command. |
| `{{article_title}}` | Article title |
| `{{original_link}}` | URL of the source article |
| `{{given_url}}` | URL as it was originally given to Wallabag |
| `{{created_at}}` | When the article was added to Wallabag |
| `{{published_at}}` | When the article was originally published, according to Wallabag |
| `{{updated_at}}` | When the article was last modified in Wallabag |
| `{{wallabag_link}}` | Link to the article in your Wallabag instance |
| `{{content}}` | Article content extracted by Wallabag (HTML, or Markdown if conversion is enabled) |
| `{{annotations}}` | All highlights as Markdown blockquotes, each followed by its annotation note if there is one |
| `{{pdf_link}}` | Vault path of the exported PDF. Only set when PDF export is enabled. |
| `{{tags}}` | Article tags, formatted according to the *Tag format* setting |
| `{{reading_time}}` | Estimated reading time in minutes |
| `{{preview_picture}}` | URL of the article's preview image |
| `{{domain_name}}` | Domain of the source article |
| `{{is_archived}}` | `true` or `false` |
| `{{is_starred}}` | `true` or `false` |

Example of a custom annotations template:

```markdown
---
wallabag_id: {{id}}
source: {{original_link}}
tags: {{tags}}
---
# {{article_title}}

{{annotations}}
```

The built-in annotations template does not include `{{pdf_link}}`. If you combine PDF export with annotations-only mode and want the link in the note, use a custom template.

### Example: reading PDFs with Annotator

Enable *Export as PDF* and *Create note*, and use the following template to open each PDF in [Annotator](https://github.com/elias-sundqvist/obsidian-annotator):

```markdown
---
annotation-target: {{pdf_link}}
---
```

![](screenshots/ss2.png)

## Data and privacy

The plugin stores its state in its own plugin folder, `<VAULT>/.obsidian/plugins/wallabag/`:

| File | Content |
| :--- | :--- |
| `data.json` | Plugin settings |
| `.synced` | JSON list of the IDs of articles that have already been synced |
| `.__wallabag_token__` | OAuth client ID, client secret, access token and refresh token, **in plain text** |

If you sync or version your vault (Git, cloud storage), consider excluding `.__wallabag_token__`. Network requests go only to the Wallabag server you configure.

## Development

```bash
npm install
npm run dev     # esbuild in watch mode
npm run build   # type check + production bundle (main.js)
npm run lint    # ESLint on src/
```

There is no automated test suite. After a change, run `npm run lint` and `npm run build`, copy `main.js`, `manifest.json` and `styles.css` into a test vault, and reload the plugin. [`AGENTS.md`](AGENTS.md) describes the code layout and conventions.

Source layout:

- `src/main.ts`: plugin bootstrap, settings and token loading, command registration
- `src/command/`: the three commands
- `src/settings/`: settings model and settings tab
- `src/wallabag/`: OAuth and Wallabag REST API calls
- `src/note/`: built-in templates and variable substitution

## Credits and license

Original plugin by [Hüseyin Zengin](https://github.com/huseyz) and contributors. Fork maintained by [chrisOTM](https://github.com/chrisOTM). Released under the [MIT License](LICENSE).
