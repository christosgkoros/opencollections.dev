# opencollections.dev

A single-file static portal for the [Open Collections](https://www.postman.com/opencollections)
Postman team. `index.html` is the whole site; Netlify publishes the repo root as-is.

## Keeping it in sync with Postman

The workspace data in `index.html` is static. It does not track Postman on its own — nothing
updates until the sync script runs and the result is pushed.

Sync is **manual by design**. Run it after you add or change collections in Postman:

- **From GitHub:** Actions → *Sync from Postman* → *Run workflow*. It commits to `main` only if
  something actually changed, and Netlify deploys on that push. Requires a `POSTMAN_API_KEY`
  repository secret.
- **Locally:**
  ```sh
  POSTMAN_API_KEY=... node scripts/sync.mjs            # writes index.html
  POSTMAN_API_KEY=... node scripts/sync.mjs --dry-run  # report only
  ```

### What the script owns, and what it doesn't

It only ever rewrites the `collections` array of each workspace, plus the static counter
literals in the markup. The curated fields — display `name`, `icon`, `category`, `description` —
are yours, and it never touches them.

The portal is a curated subset of the team's public workspaces, so the script also refuses to
guess:

- a public workspace missing from the portal is **reported, never added** (it has no icon or
  category yet, and some are excluded on purpose);
- a curated workspace that vanished from the API is **reported, never deleted** — a transient API
  error must not silently drop an entry.

Both cases show up in the run's output, along with any collections added, removed, or renamed, and
a status code for every derived workspace URL. That last check matters because
`workspaceUrl()` builds the public link from the *display* name — renaming a workspace in Postman
can break the link with no other symptom.

Descriptions are prose and never auto-edited, so when a workspace gains or loses collections the
script reminds you to re-read its description.
