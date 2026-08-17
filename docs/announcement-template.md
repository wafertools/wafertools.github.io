# Posting a release announcement

When a release is genuinely What's New-worthy, post it to
[GitHub Discussions → Announcements](https://github.com/wafertools/.github/discussions/categories/announcements)
by hand. Keep the post itself short — it's a teaser and a pointer, **not** a second copy of
the write-up. The full version lives in `whats-new.md` and only there; a duplicated narrative
in a Discussion is exactly the kind of second copy that goes stale silently, since nothing
can check a Discussion post against the doc the way `check-whats-new-freshness.yml` checks
the doc against a release.

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
