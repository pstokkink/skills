# Skills
 
This repository contains skills helpful for development work.

## Installation

Use the [skills CLI](https://github.com/vercel-labs/skills) with Node.js 22.20 or later.
Install globally to make the skills available across projects (note: this may fail for 
some agents):

```bash
npx skills add pstokkink/skills --global
```

Choose the skills and coding agents when prompted. To install every skill
explicitly, use:

```bash
npx skills add pstokkink/skills --global --skill '*'
```

You can also install a single skill:

```bash
npx skills add pstokkink/skills --global --skill bump-version
```

Omit `--global` to install into the current project instead. To list the
available skills without installing them:

```bash
npx skills add pstokkink/skills --list
```

No npm package or global CLI installation is needed; `npx` runs the installer
directly from npm, and the skills are fetched from this GitHub repository.

## Available skills

| Name | Description |
| --- | --- |
| [`/bump-version`](./bump-version/SKILL.md) | Bump a Python project version and prepare the release metadata. |
| [`/mentor`](./mentor/SKILL.md) | Guide developer-led planning before implementation and check understanding before handoff, with optional private learning history. |
