---
name: new-project
description: Scaffold a new software project from scratch. Use when the user wants to start a new project, bootstrap a repo, scaffold an app, or says "new project".
---

## Steps to take to set up a new project

- initialize a new git repo
- create LICENSE file indicating all rights reserved, not open source
- ask the language that will be used for this project, and create an appropriate gitignore
- create docs/ directory to contain docs
- create an empty architecture.md in the docs/ directory
- create an empty requirements.md in the docs/ directory
- create a scripts/ directory to contain automation scripts (like deploy, publish, etc.)
- create an AGENTS.md that contains instructions to claude/codes/antigravity
- create a CLAUDE.md that contains nothing but a single line pointing to AGENTS.md
- create a gitattributes file that specifies eol=lf for .sh files and all Dockerfile files, so that things work well when edited on windows 11
- create a .claude/skills directory
- create a secrets/ directory that will contain secrets files
- create an empty .env file and an empty .env.example file
- create a python virtual environment
- Ask if there is an upstream github repo - if there is, ask to set up the git remote to it

Important: Do each of these only if the requested artifact does not already exist - do not overwrite anything
