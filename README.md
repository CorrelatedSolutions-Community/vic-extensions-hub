# VIC Extensions & vicpyx Community Hub

A community-curated index of custom extensions and `vicpyx` scripts for
Correlated Solutions VIC software (VIC-3D, VIC-2D, and `vicpyx`).

> **Not an official Correlated Solutions product.** Everything linked here is
> contributed by the community and is **not supported by Correlated Solutions**.
> Extensions run inside your VIC environment with full access to your data and
> system — review the source before installing. Use at your own risk. See
> [DISCLAIMER](#disclaimer).

## Find something

Browse by category below, or use GitHub's search across the org. Each entry
lists the target product and the minimum version it was tested against.

### VIC-3D extensions

| Extension | What it does | Target | Min. version | Author |
|-----------|--------------|--------|--------------|--------|
| _(example)_ [cte-calculator](https://github.com/CorrelatedSolutions-Community/cte-calculator) | Computes CTE per inspector region from strain + analog temperature CSVs | VIC-3D | 11 | @example |

### VIC-2D extensions

| Extension | What it does | Target | Min. version | Author |
|-----------|--------------|--------|--------------|--------|
| _(example)_ [coordinate-rotation](https://github.com/CorrelatedSolutions-Community/coordinate-rotation) | Re-expresses full-field results in an in-plane-rotated frame | VIC-2D | 8 | @example |

### vicpyx scripts

Single-file scripts and snippets live in the
[**vicpyx-scripts**](https://github.com/CorrelatedSolutions-Community/vicpyx-scripts)
repo, organized by topic. Larger multi-file tools get their own repo and are
listed above.

## Share your work

You don't need to know git. Pick whichever path fits you:

1. **Easiest — post it.** Open a [Show & Tell discussion](https://github.com/orgs/CorrelatedSolutions-Community/discussions)
   and paste your script or attach a `.zve`/`.zip`. Describe what it does and
   which product/version it targets. A maintainer will help polish it and add it
   to this index.
2. **Standard — submit an issue.** Open a
   ["Share an extension" issue](https://github.com/CorrelatedSolutions-Community/vic-extensions-hub/issues/new/choose)
   and fill in the form.
3. **Power users — open a PR.** Add your entry to this README, or push your
   extension to its own repo using the
   [extension template](https://github.com/CorrelatedSolutions-Community/extension-template).

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full walkthrough and quality bar,
and the [Extension Authoring Guide](EXTENSION_AUTHORING_GUIDE.md) for the
hard-won technical details of building VIC extensions.

## Compatibility tags

When you publish a repo, add these as GitHub **topics** so people can filter:

- Product: `vic-3d`, `vic-2d`, `vicpyx`
- Kind: `full-field`, `inspector`, `export`, `single-processor`, `recipe`
- Version: `v8`, `v9`, `v11`, … (the major you tested against)

## Disclaimer

Community extensions are provided "as is", without warranty of any kind.
Correlated Solutions does not endorse, support, or guarantee any contribution
here. Featured entries receive a light maintainer review for obvious safety
issues, but this is **not** a security audit. Always read the source of anything
you run. Each project declares its own open-source license — check it before
reuse.
