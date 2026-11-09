---
name: aikido-mcp-deep-review
description: Runs an in-depth Aikido Deep Review of local git changes through the Aikido MCP server. An AI security review that traces data flows, auth boundaries, and business logic across the full repository, not only the diff. Takes 2 to 4 minutes and costs 1 credit per run. Use only when the user explicitly asks for a deep review, an in-depth security review, or a review of their branch or uncommitted changes. For the quick per-file scan after code edits, use aikido-mcp-scan instead.
---

This is the in-depth scan. Unlike the quick scan (aikido-mcp-scan), it reasons across the whole repository to find business-logic flaws and context-dependent vulnerabilities. Every call starts a new review and costs 1 credit, so never start one unless the user asked for it.

When running a Deep Review using the Aikido MCP server:

1. Start the review with **aikido-mcp:aikido_deep_review**:
   - Set `directory` to the absolute path of the local git repository (required).
   - Set `scope` from what the user asked for: `all` (default, commits not on the default branch plus uncommitted edits), `committed` (only commits since the merge-base), or `uncommitted` (only staged and unstaged edits on top of a pushed `HEAD`).
   - Set `base_ref` (for example `origin/main`) only if the user wants to diff against something other than the remote default branch.
   - Untracked files are skipped and listed in `skipped_untracked_files`. Set `include_untracked` if the user wants new files reviewed too. It is not allowed with scope `committed`.
2. The call returns an `agent_id` while the review runs. Call **aikido-mcp:aikido_deep_review_status** with that `agent_id`. Each call waits up to about 50 seconds. While the status is `running`, call it again (this does not start a new review or use credits). Do not call `aikido_deep_review` again to check progress.
3. Handle the final status:
   - `completed`: the findings are included. Continue to step 4.
   - `skipped`: the diff only changed files Deep Review ignores (lockfiles, generated or vendored code). Tell the user; the credit was refunded.
   - `failed`: the review could not run. Tell the user; the credit was refunded.
4. If issues are found:
   - Explain each issue clearly: title, description, severity, file location, and line numbers.
   - Apply fixes guided by the remediation provided by Aikido.
   - Verify the fixed files with the quick scan (**aikido-mcp:aikido_scan_paths**). Do not start another Deep Review to verify fixes unless the user asks, since it costs another credit.
5. Report the result to the user: all clear, or the list of findings with what was fixed and what remains.

If the Aikido MCP server is not available or fails to start, inform the user:

> The Aikido MCP server is required for Deep Review but is not available.
> Install it following the official Aikido MCP setup guide: https://help.aikido.dev/ide-plugins/aikido-mcp
