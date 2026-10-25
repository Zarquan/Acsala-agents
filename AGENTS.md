<!--
  <meta:header>
    <meta:licence>
      Copyright (C) 2026 by Wizzard Solutions Ltd, wizzard@metagrid.co.uk

      This information is free software: you can redistribute it and/or modify
      it under the terms of the GNU General Public License as published by
      the Free Software Foundation, either version 3 of the License, or
      (at your option) any later version.

      This information is distributed in the hope that it will be useful,
      but WITHOUT ANY WARRANTY; without even the implied warranty of
      MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
      GNU General Public License for more details.

      You should have received a copy of the GNU General Public License
      along with this program.  If not, see <http://www.gnu.org/licenses/>.
    </meta:licence>
  </meta:header>

  AIMetrics: [
      {
      "timestamp": "2026-10-25T17:23:56",
      "name": "@deepseek-ai/dsh",
      "version": "0.2.0-rc.2",
      "model": "deepseek-v4.1-flash-1017",
      "contribution": {
        "value": 100,
        "units": "%"
        }
      }
    ]
-->

# AGENTS.md

Guidance for AI agents working in this repository. Humans should start with
[README.md](README.md), [CONTRIBUTING.md](CONTRIBUTING.md) and
[CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## What this is

The shared agent rules, git identity hooks and GitHub tooling for Zarquan
projects. The repository is intended to become a submodule that other
projects link in, so that agent authorship, DCO sign-off and GitHub access
are handled the same way everywhere.

This repository is **not** the DSH container project. The Dockerfile, the
harness home and the session-migration tooling live in
[`Zarquan/Lithosia-docker`](https://github.com/Zarquan/Lithosia-docker).

## Layout

| Path | What it is |
|---|---|
| `agents/rules/` | The four shared rules. Copied as-is; the rule core is shared with the Calycopis projects. |
| `agents/hooks/commit-msg` | DCO guard for commits made from an agent session. |
| `agents/git-identity.env` | `AGENT_GIT_NAME` and `AGENT_GIT_EMAIL`, the author of agent commits. |
| `bin/agent-commit` | Commit wrapper: agent author, human committer, `--signoff` always applied. |
| `bin/setup-agent-git` | Sets `core.hooksPath` to `agents/hooks` for the current clone. |
| `bin/gh` | GitHub CLI wrapper that reads a token from a mounted secret file. |
| `notes/` | Dated operational notes. `20261025-02-project-split.txt` records the split from `Lithosia-docker`. |

## Current state and portability

The initial contents were copied as-is from `Lithosia-docker` at commit
`44d293c` on 2026-10-25. They still assume they sit at the root of the
consuming repository:

- `bin/agent-commit` and the `commit-msg` hook resolve `agents/` from
  `git rev-parse --show-toplevel`;
- `bin/setup-agent-git` writes an absolute `core.hooksPath`;
- `bin/gh` hard-codes the DSH/podman secret locations;
- `agents/rules/licence-header.mdc` still names the `lithosia-quadra`
  project in its frontmatter.

Do not add this repository as a submodule and expect the tools to work
until that path handling is changed. Until then, copy the files into a
project or mount them at its root. Making the tools portable is the next
piece of work, not a defect to work around silently.

## Safety rules

1. **Never commit a token or a secret.** The tokens live outside the
   repository, in `/run/secrets/`. Do not print them, paste them into a
   file, or put one on a command line.
2. **`agents/git-identity.env` is an identity, not a credential.** It holds
   the name and email recorded as the commit author and is safe to commit.
3. **Do not rewrite shared history** to change authorship or sign-off. Use
   a new commit, or ask the human first.

## Coding rules

The rules in [`agents/rules/`](agents/rules/) apply to every file an agent
creates or modifies, and to every agent commit message.

| Rule | Requirement |
|---|---|
| [`licence-header.mdc`](agents/rules/licence-header.mdc) | Every new source file starts with the GPL `<meta:header>` block, using the comment syntax for its language and the Wizzard Solutions Ltd copyright line. |
| [`copyright-year.mdc`](agents/rules/copyright-year.mdc) | When a file with a `<meta:licence>` block is modified, bump its `Copyright (C) YYYY` to the current year. |
| [`ai-metrics.mdc`](agents/rules/ai-metrics.mdc) | One `AIMetrics` entry per **change**, not per edit, appended to a file header: `timestamp` for a single pass, or `interval` covering a sequence of edits. Every agent commit message, and every GitHub issue, pull request, comment, review or release an agent posts, ends with an `AIMetrics` block in a fenced code block. |
| [`unexpected-behaviour.mdc`](agents/rules/unexpected-behaviour.mdc) | Stop and ask before coding around unexpected behaviour from an API, service or component. |

The `name`, `version` and `model` values must describe the agent that
actually did the work, not an example from the rule. For DSH that is
`@deepseek-ai/dsh`, the installed version (`dsh --version`), and the
`model` from the `agent-default-model` entry of the active profile
(`$DSH_HOME/profiles/<profile>/cordis.patch.yml`).

Headers are not required on dotfiles, lockfiles, machine-generated files,
or `agents/rules/*.mdc`. Hand-authored YAML, JSON, XML, Dockerfiles and
shell scripts are source and do get a header.

## Conventions

- **Commits**: past-tense subject, blank line, then a body explaining what
  changed and why. Keep each change focused.
- **Commit identity and DCO sign-off**: agent commits are authored by the
  agent and signed off by the human through `bin/agent-commit`. See below.
- **Licence**: GPL-3.0-or-later. New `notes/*.txt` files keep the leading
  `<meta:header>` block and the `#zrq-notes-*` tags.
- **Notes**: named `YYYYMMDD-NN-topic.txt`, one topic per file.
- **Style**: no new dependencies unless there is no alternative; comment
  the *why*; prefer a loud, early failure over a silent partial result.
- **No secrets, no generated scratch**: `agents/scratch/` is ignored.

## Commit identity and sign-off

Agent commits are made with the agent as **author** and the repository's
configured user as **committer**. `git commit --signoff` names the
committer, so the human who approves the change is the one certifying it.

| Field | Value |
|---|---|
| Author | `DeepSeek Harness <zrq-github+dsh@metagrid.co.uk>`, from [`agents/git-identity.env`](agents/git-identity.env) |
| Committer and `Signed-off-by` | your `.git/config` identity, e.g. `Zarquan <zrq-github@metagrid.co.uk>` |

```bash
bin/setup-agent-git              # once per clone, enables the DCO guard
bin/agent-commit -m "Message"
bin/agent-commit -F -            # message on stdin
```

Rules for agents:

- Commit **only after the human has explicitly approved the exact change
  set and the commit message**. That approval is the DCO certification;
  a general "looks good" is not approval to commit.
- Pass whatever `git commit` arguments you need (`-m`, `-F -`, `--amend`,
  `--allow-empty`). `--author` is rejected because the wrapper fixes it.
- `AGENT_GIT_NAME`/`AGENT_GIT_EMAIL` override the committed identity for a
  one-off; the guard resolves the identity the same way.
- The `Signed-off-by` trailer is appended after the `AIMetrics` block.

The hook refuses a commit from a DSH session that did not go through the
wrapper. It is per clone (`core.hooksPath`), and `git commit --no-verify`
bypasses it. `git merge` and `git rebase` do not run `commit-msg`, so they
are not covered.

## GitHub access

`bin/gh` reads a token from a mounted secret and passes it to the GitHub CLI
in the child process environment:

```bash
bin/gh pr create --repo Zarquan/Acsala-agents \
    --base main --head <branch> \
    --title "<title>" --body-file <file>
```

- Default token: `/run/secrets/dsh-github-token-zarquan`.
- uksrc token: `GH_SECRET_FILE=/run/secrets/dsh-github-token-uksrc bin/gh ...`,
  assigned inline on the command. Do not `export` it: DSH strips
  credential-shaped variables from spawned shells, so an exported
  `GH_SECRET_FILE` never arrives and `bin/gh` silently falls back.
- An explicit `GH_TOKEN` is passed through untouched.
- Never put a token in a URL or `-c http.extraheader`; it can be captured by
  `ps` or a transcript.
- A fine-grained token needs **Contents: Read and write** to push and
  **Pull requests: Read and write** to open a PR. The `permissions` object
  from the REST API describes the account, not the token, so it cannot be
  used to infer a token's grants.

## Provenance

The rules, hooks, identity and `bin/` scripts were copied unchanged from
`Lithosia-docker`; the agent-related notes came with them. New files in
this repository (this guide, the README, `.gitignore` and the split note's
result section) describe the new project. See
[`notes/20261025-02-project-split.txt`](notes/20261025-02-project-split.txt).
