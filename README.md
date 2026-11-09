# device-protection-skills

Agent skills for working with Aikido. Currently includes:

- **[aikido-device-protection](skills/aikido-device-protection/SKILL.md)**:
  teaches coding agents to recognize when a package install failure is actually
  a Device Protection block (not an auth or registry problem), check
  `aikidoctl events --json` before debugging, and avoid bypassing the block.
- **[aikido-mcp-comments](skills/aikido-mcp-comments/SKILL.md)**: reads the
  comments already posted on a GitHub, GitLab, or Azure DevOps pull/merge
  request (including Aikido's own scan-result comment) through the native SCM
  CLI (`gh`, `glab`, `az`) rather than the Aikido MCP server.
- **[aikido-mcp-scan](skills/aikido-mcp-scan/SKILL.md)**: quick scan of
  generated or changed code for SAST issues, exposed secrets, and IaC
  misconfigurations through the Aikido MCP server, fixes the findings, and
  rescans to confirm they are resolved.
- **[aikido-mcp-deep-review](skills/aikido-mcp-deep-review/SKILL.md)**:
  in-depth [Deep Review](https://www.aikido.dev/code/pr-review) of local git
  changes that reasons across the full repository for business-logic and
  context-dependent issues. Runs only when asked, since each run costs a credit.

## Install

Using the [skills CLI](https://www.npmjs.com/package/skills):

```bash
npx skills add noon-security/device-protection-skills
```

This clones the repo, lets you pick which agents to install to, and symlinks the
skills into the right directory (e.g. `.claude/skills/` for Claude Code).

To install without prompts:

```bash
npx skills add noon-security/device-protection-skills --all
```

To install a specific skill only:

```bash
npx skills add noon-security/device-protection-skills -s aikido-device-protection
```

## Manual install

Copy the skill directories you want from `skills/` into your agent's skills
directory (e.g. `.claude/skills/aikido-device-protection`).

## License

[Apache License 2.0](LICENSE)
