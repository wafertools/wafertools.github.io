# Posting a release announcement

When a release is genuinely What's New-worthy, post it to
[GitHub Discussions → Announcements](https://github.com/wafertools/.github/discussions/categories/announcements)
by hand. Keep the post itself short — it's a teaser and a pointer, **not** a second copy of
the write-up. The full version lives in `whats-new.md` and only there; a duplicated narrative
in a Discussion is exactly the kind of second copy that goes stale silently, since nothing
can check a Discussion post against the doc the way `check-whats-new-freshness.yml` checks
the doc against a release.

## At every release

1. Add the entry to `whats-new.md` (that is the write-up, and the only copy of it).
2. Post the teaser below to Announcements.
3. **Check the existing Discussion posts for sentences that have since decayed.** Nothing
   automated can do this — `check-whats-new-freshness.yml` compares the *doc* against the
   sibling changelogs and reads no Discussions at all.

   Known case: [Discussion #2](https://github.com/wafertools/.github/discussions/2)
   ("Introducing What's New") carries a **Latest entry:** paragraph naming the 2026-08-16
   entry. **The fix is to delete that paragraph, not to keep updating it** — a running-status
   sentence in a place nothing can check will decay again on the next release, and re-editing
   it every time is a maintenance task that exists only because the sentence should not have
   been written. Once it is gone, this step becomes a quick scan rather than a chore.

## Never write a sentence that decays

The teaser says what changed *in that release*. It must not contain a claim that silently
goes stale as soon as the next release ships — no "latest entry", no "now supports X" framed
as a running status, no version number presented as current.

This has already happened: [Discussion #2](https://github.com/wafertools/.github/discussions/2)
("Introducing What's New") carries a **Latest entry:** paragraph naming the 2026-08-16 entry.
It was accurate the day it was posted and wrong the moment 2026-08-23 shipped, and nothing
can catch it — `check-whats-new-freshness.yml` compares the doc against the sibling
CHANGELOGs, and no check reads Discussions at all. That is the same "second copy goes stale
silently" failure this page warns about, in the one post that introduced the page.

A post is a dated snapshot. Write it so it stays true as a record of that date, and let
`whats-new.md` carry the current state.

## Template

```
Title: <one line, plain language — matches the whats-new.md entry's own heading>

<1–2 sentence teaser. What changed, why someone would care. No implementation detail.>

Full write-up: https://wafertools.github.io/whats-new/#<anchor-of-the-entry>
```

## Example

```
Title: Filter large batches before loading; wafer maps handle data with no position info

tsmap can now scan a whole directory of files for their metadata before loading anything,
and wafermap now handles test data with no die position at all instead of dropping it.

Full write-up: https://wafertools.github.io/whats-new/#2026-08-16-filter-large-batches-before-loading-wafer-maps-handle-data-with-no-position-info
```
