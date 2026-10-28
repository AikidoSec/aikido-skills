# Fetching PR/MR comments per platform

## GitHub (`gh`)

- **Install**: `brew install gh` (or see https://cli.github.com for other package managers).
- **Auth check** (only after a failed fetch): `gh auth login`.
- **Fetch**, current branch's PR: `gh pr view --comments`
- **Fetch**, specific PR: `gh pr view <number> --comments` (add `--repo org/repo` for a repo
  that isn't the current working directory).
- **Fetch inline review comments** too: `--comments` leaves out line-level review comments,
  which is where bots often post findings. Run this as well:
  ```
  gh api repos/{owner}/{repo}/pulls/<number>/comments --paginate \
    --jq '.[] | "\(.user.login) \(.created_at) \(.path):\(.line // .original_line)\n\(.body)\n---"'
  ```
  `{owner}` and `{repo}` fill in from the current directory. For another repo, write them out.

## GitLab (`glab`)

- **Install**: `brew install glab` (or see https://gitlab.com/gitlab-org/cli for other package
  managers). For a self-managed instance, also set `GITLAB_HOST=<your-host>` or pass
  `--hostname <your-host>`.
- **Auth check** (only after a failed fetch): `glab auth login`.
- **Fetch**, current branch's MR: `glab mr view --comments`
- **Fetch**, specific MR: `glab mr view <number> --comments` (add `--repo org/repo` for a repo
  that isn't the current working directory).

## Azure DevOps (`az`)

There's no dedicated `az repos pr comment`/`view --comments` command, so this uses the generic
REST invoker instead.

- **Install**: `brew install azure-cli`, then `az extension add --name azure-devops`.
- **Auth check** (only after a failed fetch): `az login` (or `az devops configure --defaults
  organization=<org-url> project=<project>` if defaults aren't set).
- **Fetch** PR comment threads:
  ```
  az devops invoke \
    --area git --resource pullRequestThreads \
    --route-parameters project=<project> repositoryId=<repo-id> pullRequestId=<pr-id> \
    --organization <org-url> \
    --api-version 7.1
  ```
  This returns raw JSON: each thread has a `comments` array with `content` and `author`.
  Parse it directly rather than looking for a prettier flag, none exists.

## Bitbucket (`bkt`)

Atlassian has no official Bitbucket CLI. `bkt` (https://github.com/avivsinai/bitbucket-cli) is
community-maintained and covers both Bitbucket Cloud and Data Center.

- **Install**: `brew install bitbucket-cli` (installs the `bkt` binary; see the repo for other
  package managers).
- **Auth check** (only after a failed fetch):
  - Cloud: `bkt auth login https://bitbucket.org --kind cloud --web`, then
    `bkt context create cloud --host api.bitbucket.org --workspace <workspace> --set-active`.
  - Data Center: `bkt auth login https://<your-host> --web-token`, then
    `bkt context create dc --host <your-host> --project <project-key> --set-active`.
- **Pick the context** for the remote first: `bkt` uses the active context's server, and only
  reads the repo from the git remote when that server matches. Otherwise it silently falls back
  to the context's saved repo. Run `bkt context list --json` and take the `name` whose `host`
  matches the remote: `api.bitbucket.org` for a `bitbucket.org` remote, the server hostname for
  Data Center. Pass it as `-c <name>` on every `bkt` command below. No match means no context,
  see the recovery step in SKILL.md.
- **Find the PR ID** for the current branch: `bkt pr comments` needs a PR number, it doesn't
  detect the current branch's PR. Look it up first:
  ```
  bkt pr list -c <name> --state OPEN --limit 0 --json \
    --jq '.pull_requests[] | select((.source.branch.name // .fromRef.displayId) == "<branch>") | .id'
  ```
  `<branch>` is the output of `git branch --show-current`. Cloud returns `source.branch.name`,
  Data Center returns `fromRef.displayId`, the filter handles both. If it prints nothing,
  `null`, or more than one ID, ask the user for the PR number.
- **Fetch**, specific PR: `bkt pr comments <id> -c <name> --json`. Use `--json`, the text output
  has no timestamps and drops deeply nested Data Center replies. The `comments` array holds
  general, inline and reply comments:
  - Cloud: `user.display_name`, `created_on`, `content.raw`, and `inline.path`/`inline.to` for
    inline comments.
  - Data Center: `author.displayName`, `createdDate` (epoch milliseconds), `text`, and
    `anchor.path`/`anchor.line` for inline comments.

  For a repo that isn't the current working directory, add `--repo <slug>` plus
  `--workspace <workspace>` (Cloud) or `--project <key>` (Data Center).
