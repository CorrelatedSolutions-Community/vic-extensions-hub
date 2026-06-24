# Contributing

Thanks for sharing your work with the VIC community! This guide covers what to
submit, how, and the quality bar for getting featured in the hub index.

## Three ways to contribute (pick one)

| Path | Best for | What you do |
|------|----------|-------------|
| **Show & Tell** | "I made a thing, here it is" | Post in [Discussions](https://github.com/orgs/CorrelatedSolutions-Community/discussions), attach the file. A maintainer takes it from there. |
| **Submit an issue** | A finished extension you want indexed | Open a ["Share an extension" issue](../../issues/new/choose) and fill in the form. |
| **Open a PR** | You're comfortable with git | Use the [extension template](https://github.com/CorrelatedSolutions-Community/extension-template), push your repo, then add a row to the hub README. |

You never need git to contribute — Show & Tell and issues both work from the
browser.

## What makes a good contribution

A maintainer will feature your extension in the index when it has:

- [ ] **A clear README** — what it does, which product (VIC-3D / VIC-2D /
  vicpyx) and **minimum version** you tested against, install steps, and ideally
  a screenshot or example output.
- [ ] **A license.** Declare one (we recommend **MIT** or **Apache-2.0**). No
  license = we can't feature it, because nobody legally knows if they can use it.
- [ ] **Self-contained install.** A packaged `.zve` for extensions, or a single
  `.py` (plus a `requirements.txt` if it needs packages) for scripts.
- [ ] **No bundled data dumps or secrets.** Strip customer data, absolute paths,
  and credentials. Use a small synthetic sample if you need example data.
- [ ] **Readable source.** Others will read it before they run it — favor pure,
  testable functions over one giant `process_data`.

## Licensing

Add a `LICENSE` file to your repo. If you're unsure, **MIT** is the simplest
permissive choice. By contributing you confirm you have the right to share the
code and agree it's distributed under the license you declare.

## Safety review

Because extensions run inside the user's VIC environment with full file/system
access, maintainers do a **light review** of anything featured in the index for
obvious red flags (network calls, file deletion, obfuscated code). This is not a
security audit and confers no guarantee — see the README disclaimer. Don't
submit code you wouldn't run yourself.

## Naming & tags

- Repo name: lowercase-with-hyphens, descriptive (`weld-seam-tracker`, not
  `vic-tool-3`).
- Add GitHub **topics** for product, kind, and version (see the hub README).

## Questions

Open a discussion in the [Q&A category](https://github.com/orgs/CorrelatedSolutions-Community/discussions).
For how to actually build an extension, read the
[Extension Authoring Guide](EXTENSION_AUTHORING_GUIDE.md).
