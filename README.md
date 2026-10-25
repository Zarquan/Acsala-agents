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

# Acsala agents

Common agent rules, git identity hooks and GitHub tooling for Zarquan
projects. The eventual aim is a single reusable source that other projects
can bring in as a git submodule, so every project records agent
contributions and human sign-off in the same way.

This project is named after
[*Acsala anomala*](https://en.wikipedia.org/wiki/Acsala), a lichen moth in
the monotypic genus *Acsala* (subtribe Acsalina, family Erebidae), known
from Alaska and Yukon.

<a title="Acsala anomala CSU ENT1014791, CSU Department of Bioagricultural Sciences and Pest Management, CC0 1.0, via Wikimedia Commons" href="https://en.wikipedia.org/wiki/Acsala#/media/File:Acsala_anomala_CSU_ENT1014791.jpg"><img width="640" alt="Acsala anomala moth" src="https://upload.wikimedia.org/wikipedia/commons/9/99/Acsala_anomala_CSU_ENT1014791.jpg"></a>

*[Acsala anomala CSU ENT1014791.jpg](https://commons.wikimedia.org/wiki/File:Acsala_anomala_CSU_ENT1014791.jpg)
by CSU, Department of Bioagricultural Sciences and Pest Management, via
Wikimedia Commons, dedicated to the public domain under
[CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).*

## What it provides

| Path | What it is |
|---|---|
| [`agents/rules/`](agents/rules/) | Shared coding rules: licence headers, copyright year, AI metrics and unexpected behaviour. |
| [`agents/hooks/commit-msg`](agents/hooks/commit-msg) | DCO guard for commits made inside an agent session. |
| [`agents/git-identity.env`](agents/git-identity.env) | The identity recorded as the author of agent commits. |
| [`bin/agent-commit`](bin/agent-commit) | Commit wrapper: agent as author, repository user as committer and `Signed-off-by`. |
| [`bin/setup-agent-git`](bin/setup-agent-git) | Enables the versioned hooks in a clone. |
| [`bin/gh`](bin/gh) | GitHub CLI wrapper that reads a token from a mounted secret file. |
| [`notes/`](notes/) | Dated operational notes, including the provenance of this split. |

## Using it in a project

The code still assumes it lives at the root of the consuming repository.
In particular, `bin/agent-commit`, `bin/setup-agent-git` and the hook
resolve `agents/` from `git rev-parse --show-toplevel`. Until the
portability work is done, use the files by copying them into a project, or
mount this tree at the project root; adding it as a submodule will require
the path changes described under [Status](#status).

Typical use in a clone that has the files at its root:

```bash
# Once per clone: point core.hooksPath at agents/hooks.
bin/setup-agent-git

# Commit with the agent as author and the human as committer.
bin/agent-commit -m "Describe the change"

# Use the GitHub CLI with the mounted token.
bin/gh pr create --repo Zarquan/Acsala-agents \
    --base main --head <branch> \
    --title "<title>" --body-file <file>
```

`bin/gh` reads `/run/secrets/dsh-github-token-zarquan` by default and
accepts `GH_SECRET_FILE` for the uksrc token, or an explicit `GH_TOKEN`.
See [AGENTS.md](AGENTS.md) for the full token and commit rules.

## Status

This repository starts as an as-is copy of the non-DSH source from
[`Zarquan/Lithosia-docker`](https://github.com/Zarquan/Lithosia-docker):
the agent rules, hooks, git identity and `bin/` tooling, plus the notes
that describe them. The initial copy was taken from commit `44d293c` on
2026-10-25.

`Lithosia-docker` keeps its own copies for now and continues to specialise
in the DSH container, the Dockerfile and the migration tooling. Later work
will:

- make the paths submodule-aware, so the tools work from a checkout under
  another project rather than only from a repository root;
- remove Lithosia-specific wording from the rule frontmatter;
- drop the DSH- and podman-specific assumptions from `bin/gh` and the
  hook, or make them configurable;
- remove the duplicated copies from `Lithosia-docker` and link this
  repository in as a submodule.

## Licence

GPL-3.0-or-later. See [LICENSE](LICENSE). Contributions are accepted under
the Developer Certificate of Origin; see [CONTRIBUTING.md](CONTRIBUTING.md)
and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
