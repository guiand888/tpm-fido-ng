# Internal Development Standards & AI Rules

This is a private Git submodule containing all internal development standards, AI rules, and proprietary instructions for the tpm-fido-ng project.

## Structure

- `rules/` - Development standards and guidelines
- `instructions/` - AI-specific instructions and custom rules

## Usage

This submodule is mounted at `dev/` in the main repository. To clone the full repository including this submodule:

```bash
git clone --recurse-submodules <repository-url>
```

Or if you've already cloned without submodules:

```bash
git submodule update --init --recursive
```

## Important

⚠️ **This directory contains proprietary content.** Do not commit this to public repositories or share without authorization.

## Adding Content

To add new rules or instructions:

1. Create files in the appropriate subdirectory (`rules/` or `instructions/`)
2. Commit changes within this submodule
3. Push to the submodule's remote repository
4. Update the submodule pointer in the main repository
