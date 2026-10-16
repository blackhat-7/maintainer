# maintainer

A strict Claude maintainer for any GitHub repo. One command to install, the same command to update.

- **Standards** for every AI and human working on the repo (`.maintainer/STANDARDS.md`), loaded through `CLAUDE.md` and `AGENTS.md`.
- **PR review** on every PR, run on your Claude subscription. Claude reads the change like a principal engineer. It closes unwanted ideas at once, requests changes on fixable ones, and approves the rest. Approved PRs from collaborators auto-merge. PRs left with changes requested close after 21 idle days.
- **PR template** and an `open-pr` skill, so AI-written PRs arrive in the right format.

## Install

Needs [`gh`](https://cli.github.com), logged in as a repo admin. In the repo:

```sh
curl -fsSL https://raw.githubusercontent.com/blackhat-7/maintainer/main/install | bash
claude setup-token                          # copy the token it prints
gh secret set CLAUDE_CODE_OAUTH_TOKEN       # paste it
```

Then commit and push the changes.

## Update

Rerun the install command. It overwrites the managed files and leaves the rest of your repo alone.

## What it sets up

| Where | What |
|---|---|
| `.github/workflows/maintainer.yml` | Reviews each PR and runs the daily stale check. Pinned to a release of this repo. |
| `.github/pull_request_template.md` | What / Why / Testing. |
| `.maintainer/` | Standards and review rules. Edit the review rules here to tune the reviewer. |
| `.claude/skills/open-pr/` | Self-review and open a PR in the right format. |
| `CLAUDE.md`, `AGENTS.md`, `CODEOWNERS` | A marked block. Your own content is kept. |
| Repo settings | Auto-merge on. Actions may approve PRs. `changes-requested` and `needs-maintainer` labels. A `maintainer` ruleset: PRs need one approval, plus yours for `.github/`, `.claude/` and `.maintainer/`. Admins can bypass. |

Make your CI a required status check in the ruleset, so auto-merge waits for it.

## Security

PRs are untrusted, including ones from forks. The review never runs PR code. Claude only gets read-only file tools, confined to the PR checkout, and it ignores the PR's own Claude config. It can't reach your token or the GitHub token. Its only output is a verdict that a fixed script acts on.

Limits that a prompt injection can't get past:

- PRs from outsiders are never auto-merged.
- Changes to `.github/`, `.claude/` or `.maintainer/` by anyone but the owner are rejected before Claude sees them.
- The merge is pinned to the reviewed commit.

Limits, checked before Claude runs so they cost nothing:

| | Outsiders | Collaborators | Owner |
|---|---|---|---|
| Changed lines (lockfiles excluded) | 1,500 | 5,000 | 5,000 |
| Claude reviews per PR | 10 | 10 | no limit |
| Open PRs at once | 3 | no limit | no limit |

## Releasing

Change files under `template/` or `.github/workflows/`, bump `VERSION` in `install`, commit, then tag the commit with that version.
