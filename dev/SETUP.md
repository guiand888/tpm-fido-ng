# Dev Submodule Setup Guide

This directory is configured as a Git submodule to store internal development standards, AI rules, and proprietary instructions.

## Initial Setup (One-Time)

If you're setting up this submodule for the first time locally, initialize it as a git repository:

```bash
cd dev
git init
git config user.email "your-email@example.com"
git config user.name "Your Name"
git add .
git commit -m "Initial dev submodule structure"
```

## Directory Structure

```
dev/
├── README.md           # Submodule documentation
├── SETUP.md           # This file
├── .gitignore         # Git ignore rules for this submodule
├── rules/             # Development standards and guidelines
│   └── .gitkeep       # Placeholder for rule files
└── instructions/      # AI-specific instructions and custom rules
    └── .gitkeep       # Placeholder for instruction files
```

## Adding Content

### Adding Rules

Place development standards and guidelines in `rules/`:

```bash
# Example: Add a new rule file
cp /path/to/my-standard.md dev/rules/
cd dev
git add rules/my-standard.md
git commit -m "Add my-standard.md"
```

### Adding Instructions

Place AI instructions and custom rules in `instructions/`:

```bash
# Example: Add a new instruction file
cp /path/to/my-instructions.md dev/instructions/
cd dev
git add instructions/my-instructions.md
git commit -m "Add my-instructions.md"
```

## Cloning with Submodule

To clone the main repository with this submodule included:

```bash
git clone --recurse-submodules <repository-url>
```

Or if already cloned without submodules:

```bash
git submodule update --init --recursive
```

## Updating Submodule Pointer

After making changes in the `dev/` submodule:

1. Commit changes within the submodule
2. Return to the main repository root
3. Stage the submodule pointer update:

```bash
cd /path/to/main/repo
git add dev
git commit -m "Update dev submodule pointer"
```

## Important Notes

⚠️ **This directory contains proprietary content.** Do not commit to public repositories.

⚠️ **Never commit private keys or secrets** to this submodule.

⚠️ **Always use `--recurse-submodules`** when cloning to ensure this directory is included.
