---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Lab 0 — Course Orientation & Environment

**Week 0 · Orientation · No AWS account**
Welcome to SAM10. This lab gets your Linux VM + shell ready and checks out this book.

## Before you begin
- A Linux VM (Ubuntu 22.04 or Amazon Linux 2) or WSL2 — the book builds with JupyterBook
  (`jupyter-book build .`).
- A browser for AWS Skill Builder (sign-in link provided week 1).

## Hands-on activity (Linux foundation)
```bash
# Verify your toolchain
bash --version | head -1        # GNU Bash 5+
python3 --version
git clone <this-repo-url> lab0  # clone the book repo
cd lab0
# build the book locally (no warnings expected)
python3 -m venv .venv && source .venv/bin/activate
pip install jupyter-book sphinx-design
jupyter-book build . --path-output .
```

## Linux skills checklist
- [ ] Navigate with `ls`, `cd`, `find`, `grep` without `-R`.
- [ ] Read one page of a man page (`man ls | head`).
- [ ] Edit a file with `vim`/`nano` (save + exit).

## Artifact
`screenshot` of this book opening at the table of contents — submit as `lab0-orientation.png`.

## Sources
- GNU Bash manual: https://www.gnu.org/software/bash/manual/
- JupyterBook docs: https://jupyterbook.org/
