# disappearing ink

A Jekyll blog published through GitHub Pages, documenting caregiving for
someone with Alzheimer's. Every entry is stamped with the number of days since
the diagnosis on **July 16, 2026** — a number the site calculates, never one
you type.

This file assumes no prior context. Anyone should be able to clone the repo and
have the site running locally in about five minutes.

---

## Table of contents

1. [Writing a new entry](#writing-a-new-entry)
2. [Front matter reference](#front-matter-reference)
3. [Publishing](#publishing)
4. [Local setup, from scratch](#local-setup-from-scratch)
5. [How the day count works](#how-the-day-count-works)
6. [Project layout](#project-layout)
7. [Changing the design](#changing-the-design)
8. [Things worth knowing](#things-worth-knowing)

---

## Writing a new entry

```bash
bin/new-post "The morning he asked about the dog"
```

That creates a file in `_posts/` dated today, fills in the front matter, and
prints the day number the entry will publish as. Open the file, write below the
front matter in Markdown, save.

To write about a day that has already passed, give the date:

```bash
bin/new-post --date 2026-08-12 "Something from last week"
```

The flag has to come before the title.

To preview before publishing:

```bash
bin/serve
```

Then open <http://localhost:4000/disappearing-ink/>. The path matters: the site
publishes under a subpath, so `baseurl` is set and the bare root will 404. The
page reloads as you save. Stop it with Ctrl-C. Anything in a `_drafts/` folder
shows up in the preview but is never published, so that's the place for an entry
you aren't sure about.

You can also skip the script entirely and create the file by hand. The filename
has to be `YYYY-MM-DD-some-slug.md` — Jekyll reads the date from the filename.

---

## Front matter reference

The block between the `---` lines at the top of an entry. Only two lines are
required:

```yaml
---
layout: post
title: "The morning he asked about the dog"
---
```

The optional ones:

| Key | Effect |
| --- | --- |
| `subtitle:` | An italic deck line under the title, on the homepage and the entry page. |
| `day:` | Overrides the calculated day number. For when you publish on a Thursday about something that happened on a Sunday and want the number to follow the story. |
| `song:` | A song the entry reminds you of, set at the foot of the page as a link out. See below. |

### Adding a song

An entry can end with one song — a link, never an embedded player:

```yaml
---
layout: post
title: "I know the end"
song:
  title: "I Know the End"
  artist: "Phoebe Bridgers"
  url: "https://www.youtube.com/watch?v=WJ9-xN6dCW4"
---
```

`title` and `url` are both required for the block to appear; `artist` is
optional and the block reads fine without it. `bin/new-post` writes these lines
into every new entry already commented out, so adding a song means deleting
four `#` characters.

Leaving the key out entirely is the normal case, and the entry then ends after
its last paragraph with nothing extra on the page.

The song appears only on the entry page — not in the homepage or archive lists,
where it would read as metadata rather than as the closing note it is.

Any URL works. The screen-reader label names YouTube specifically when the link
points there, and otherwise just says the link opens in a new tab.

Everything else — the day count, the date line, the reading-time estimate, the
homepage excerpt, the previous/next links — is derived. There is nothing else
to keep in sync.

The excerpt on the homepage is the first paragraph, trimmed to about 44 words.
To control it, make the first paragraph the one you want shown.

---

## Publishing

```bash
git add .
git commit -m "Day 34"
git push
```

GitHub rebuilds the site and the entry is live in about a minute. There is no
build step to run and no deploy workflow to babysit — GitHub Pages runs Jekyll
on its own servers.

### First-time repo setup

The site publishes at **`https://sstirling.github.io/disappearing-ink/`**, as a
project site from the existing `sstirling/disappearing-ink` repo. The root
address `https://sstirling.github.io` is not available: it already serves the
portfolio site from a repo of that name.

Because it publishes under a subpath, `baseurl` in `_config.yml` is
`"/disappearing-ink"` and must stay identical to the repo name. Renaming the
repo without changing `baseurl` leaves a site that builds cleanly and then loads
with no CSS and dead links on every page.

Two steps, both on github.com, both one-time:

1. **Make the repo public.** Settings → General → Danger Zone → Change
   visibility. GitHub Pages does not publish from a private repo on the free
   plan. This is what puts the entries in front of readers and in front of
   search engines, so it is the step to be deliberate about.
2. **Turn Pages on.** Settings → Pages → Build and deployment → Source: Deploy
   from a branch, branch `main`, folder `/ (root)`. Save.

The first build takes a few minutes; after that it's under a minute per push.
Watch it under the repo's Actions tab — a failed build shows up there rather
than as a broken page.

### Moving the site somewhere else

`url` and `baseurl` in `_config.yml` are the only place the address is
declared, and the comment above them spells out the three cases. For a custom
domain, set `url` to the domain, leave `baseurl` empty, add a file named
`CNAME` at the repo root containing just the bare domain, and point the DNS at
GitHub (an `ALIAS`/`ANAME` record to `sstirling.github.io`, or four `A` records
to `185.199.108.153`, `.109.153`, `.110.153`, `.111.153`).

---

## Local setup, from scratch

You only need this to preview locally. Publishing works without it.

macOS ships Ruby 2.6, which is too old for Jekyll. The current Homebrew default
is Ruby 4, which is too *new* — it removed a method that the Liquid version
GitHub Pages builds with still calls, so the build dies on the first template.
Ruby 3.3 is the version in between.

```bash
brew install ruby@3.3
bin/serve
```

`bin/serve` finds Ruby 3.3, installs the gems into `vendor/bundle` on first run,
and starts the preview. It does not change your shell's default Ruby, so
nothing else on your machine is affected.

If you'd rather run the Jekyll commands directly:

```bash
export PATH="$(brew --prefix ruby@3.3)/bin:$PATH"
bundle install
bundle exec jekyll serve
```

---

## How the day count works

`_config.yml` holds one line:

```yaml
diagnosis_date: 2026-07-16
```

`_includes/day-count.html` is the only file that does arithmetic with it. The
homepage, the entry pages and the archive all call that include, so changing
the date in `_config.yml` renumbers the entire archive at once and nothing can
fall out of step.

Three cases are handled:

- **After the diagnosis** — `Day 34`.
- **The diagnosis itself** — `Day 0`, with a small "diagnosis" note beneath.
- **Before the diagnosis** — `Day −12`, using a typographic minus. Useful if
  you go back and write about the months when something was clearly wrong but
  nobody had said the word yet.

Two details in that file are deliberate and worth not "simplifying" later:

**Both dates are normalized to noon before being converted to seconds.** A
midnight-to-midnight span that crosses a daylight-saving change comes out an
hour short, and integer division would silently round it down to the wrong day.

**Half a day is added before dividing.** That turns truncation into rounding,
which is what makes the result correct for dates before the diagnosis as well
as after. Verified: July 16, 2026 → Jan. 15, 2027 renders as day 183, crossing
two daylight-saving changes.

`bin/new-post` reproduces the same arithmetic in shell so it can print the day
number when it creates a file. It reads `diagnosis_date` out of `_config.yml`
rather than keeping its own copy.

---

## Project layout

```
_config.yml               site settings, diagnosis_date, timezone
Gemfile                   pins the toolchain GitHub Pages uses
index.html                homepage: entries newest first
about.md                  about page (held back: published: false)
archive.md                every entry, grouped by month

_layouts/
  default.html            page shell: head, masthead, footer
  post.html               an entry
  page.html               archive, and the about page when it returns

_includes/
  head.html               meta tags, fonts, favicon
  day-count.html          the only place day arithmetic happens
  ap-date.html            AP-style dates (Aug. 19 but July 4)
  entry.html              one entry in a list, shared by home and archive
  song.html               the optional song at the foot of an entry

assets/css/style.css      all styling, one file, commented by section
bin/new-post              creates an entry
bin/serve                 local preview
_posts/                   the entries
```

`_site/` is the built output. It is regenerated on every build and is not
committed.

---

## Changing the design

Everything visual is in `assets/css/style.css`, which is plain CSS on purpose —
Jekyll can compile Sass, but the version GitHub Pages ships is libsass, which
chokes on `clamp()`, `min()` and `color-mix()`. What you edit is what ships.

The colors, fonts and measurements are all custom properties in the `:root`
block at the top. Changing a value there changes it everywhere:

| Property | What it controls |
| --- | --- |
| `--paper`, `--ink` | Background and body text |
| `--ink-muted` | Dates, reading times, captions |
| `--accent` | Links and the day numeral |
| `--measure` | Width of the text column (currently 34rem, about 64 characters) |
| `--rail-width`, `--rail-gap` | The day marker's column in the left margin |
| `--fading-rule` | The gradient used for every dissolving hairline |

The dark palette is the `@media (prefers-color-scheme: dark)` block directly
below, and follows the reader's system setting.

Two constraints worth keeping:

**Contrast.** Every text color clears WCAG AA against its background — measured
at 16.25:1, 5.89:1 and 8.53:1 in light, and 14.55:1, 6.53:1 and 9.71:1 in dark.
The measured ratios are noted in comments beside each value. If you change a
color, re-check it.

**The day rail's breakpoint.** The rail only moves into the left margin at
1000px and up, because below roughly 940px there isn't room for the text
measure plus the rail, and the numeral gets clipped off the screen edge. If you
widen `--measure` or `--rail-width`, raise that breakpoint to match.

---

## Things worth knowing

**Day 0 is pinned to the top of the homepage.** A reader arriving cold should
land on the entry that explains who Carol is before anything else, so the entry
dated the same day as `diagnosis_date` leads the list however old it gets.
Everything below it runs newest first. It carries a small "Start here" marker on
its meta line — without one, an old entry sitting above newer ones just reads as
a broken sort. To drop the marker, delete the `include.pinned` block in
`_includes/entry.html`; to drop the pin entirely, replace the homepage's loop in
`index.html` with a plain `for post in site.posts`.

Nothing declares the pin in the post itself. It comes from `diagnosis_date`, the
same value the day count uses, so moving the date moves the pin with it. If no
entry falls on the diagnosis date, nothing is pinned and the homepage is simply
reverse-chronological.

The archive is unaffected — it groups by month and stays chronological, because
it's an index rather than a reading order.

**The About page is held back, not deleted.** `about.md` still contains the
full scaffold -- a marked draft note and bracketed placeholder text -- but
carries `published: false`, so Jekyll skips it and `/about/` is never built. The
masthead link is commented out in `_layouts/default.html` for the same reason: a
link to an unbuilt page is a 404. To bring it back, write the page, delete the
`published: false` line, and restore the link the comment there spells out.

**Future-dated entries do publish.** Jekyll hides them by default; this site
sets `future: true` in `_config.yml` because a mistyped year making an entry
vanish with no error is worse than a wrong date looking wrong. `bin/new-post`
prints the day number when it creates a file, which catches most date typos.

**Dates use AP style.** `Aug. 19, 2026` but `July 4, 2026` — months of six or
more letters get abbreviated with a specific date. Handled by
`_includes/ap-date.html`.

**There's no RSS feed and no tags.** Both are small later additions. For a
feed: add `plugins: [jekyll-feed]` to `_config.yml` and `{% feed_meta %}` to
`_includes/head.html`; the plugin is already available on GitHub Pages, so
nothing needs installing.

**No comments and no third-party trackers.** The site makes two external
requests: Google Fonts for the Newsreader typeface, and GoatCounter for visit
counts. To drop the fonts, remove the two `preconnect` lines and the
`fonts.googleapis.com` stylesheet from `_includes/head.html` — the CSS already
falls back to Georgia.

**Analytics are GoatCounter, and only on the live site.** The script sits at the
bottom of `_layouts/default.html` behind an `if jekyll.environment ==
"production"` guard, so `bin/serve` never counts your own previews as readers —
hits cannot be deleted once recorded. GitHub Pages builds with
`JEKYLL_ENV=production`, so the published site gets it automatically. To see it
locally, run `JEKYLL_ENV=production bundle exec jekyll serve`. Stats live at
<https://sstirling.goatcounter.com>. GoatCounter sets no cookies and collects no
personal data, which is why there's no consent banner; removing analytics
entirely means deleting that one block from the layout.
