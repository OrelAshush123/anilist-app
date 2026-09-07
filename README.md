# Rewatch — AniList Rating Review

A single-file web app that walks you through your AniList anime list, one title at a time, so you can quickly review and update your scores.

Live at: https://orelashush123.github.io/anilist-app/

## About the project

If you've been on AniList for a while, your old scores start to drift — a 7 you gave years ago might deserve an 9 today, or vice versa. Rewatch turns that cleanup into a fast, focused loop instead of digging through list views and edit modals: it pulls every anime you've already rated, shows them one by one with the cover art and your current score front and center, and lets you keep or update the rating with a single keystroke. Login goes through AniList's own OAuth screen — this app never sees or asks for your password — and there's no backend at all; it's one static HTML file that talks directly to AniList's GraphQL API from your browser.

## Features

- **One-tap login** — redirects to AniList, comes straight back signed in, no tokens to copy or paste
- **Review one anime at a time** — cover image, title, current score, and an editable field for the new one
- **Keyboard-first** — press Enter to save a change and move on, or press Enter unchanged to skip
- **Respects your scoring scale** — automatically adapts to whichever format your AniList account uses (100-point, 10-point, 10-point decimal, 5-star, or 3-point smiley scale)
- **Progress indicator** — always shows "X of Y" so you know how far you are
- **Handles the edge cases** — empty lists, missing titles or cover art, expired sessions, rate limiting, and failed saves (with retry) without losing your place
- **No backend, no build step** — one HTML file, hosted as a static site

## Setup

### 1. Configure the AniList API client

This app uses AniList Client ID `50350`. In [AniList's developer settings](https://anilist.co/settings/developer), that client's **Redirect URL** must be set to exactly:

```
https://orelashush123.github.io/anilist-app/
```

(Trailing slash included — AniList matches this URL exactly.)

If you fork this repo and host it elsewhere, register your own client at the link above and update:
- `CLIENT_ID` in the `<script>` section of `index.html`
- `REDIRECT_URI` in the same file, matching wherever you deploy it
- The Redirect URL in your AniList client's settings, to match

### 2. Deploy

This is a static file — any static host works. For GitHub Pages:

1. Push `index.html` to this repository.
2. In **Settings → Pages**, set the source to the branch/folder containing it.
3. Visit the published URL — it should match `REDIRECT_URI` above.

No dependencies, no build step, no environment variables.

## Usage

1. Open the site and click **Log in with AniList**.
2. Approve access on AniList's authorization page — you're redirected straight back, already signed in.
3. Click **Start updating**.
4. For each anime shown:
   - Press **Enter** with the score unchanged to skip to the next one.
   - Edit the score and press **Enter** to save it to AniList and move on.
   - Use **Skip** to move on without checking the value, or **Stop reviewing** to pause anytime — anything already saved stays saved on AniList.

## How it works

- **Auth**: AniList's implicit OAuth2 grant (`response_type=token`). The access token comes back in the redirect URL's fragment, is read out automatically, and is stored in `localStorage` only — it's never sent anywhere but AniList's own API.
- **Fetching your list**: `MediaListCollection(userId, type: ANIME)` query, flattened across all list statuses, filtered to entries with a score greater than 0.
- **Saving a score**: the `SaveMediaListEntry(id, score)` mutation. The `score` field is interpreted by AniList according to your account's `scoreFormat`, which the app also uses to validate input (integer vs. decimal, correct min/max) before submitting.

## Privacy & security

- No backend, no analytics, no third-party services beyond AniList's own API.
- Your AniList password is never entered into this app.
- The access token lives only in your browser's `localStorage`, scoped to this site.
