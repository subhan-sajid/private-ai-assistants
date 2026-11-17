# Build Log

Newest entries at the top. Every entry: what I did, what went wrong, how I fixed it, what I learned.

---

## 2026-11-17: Repo and task board on GitHub

**Did**
- Logged the GitHub CLI in (`gh auth login --scopes project`).
- Made the first commit and created this public repo with `gh repo create --public --source . --push`.
- Created a public GitHub Project board with one issue per phase (#1–#10). Phase 0 is *In Progress* and labelled `blocked` until AWS sign-in works.

**Mistake**
- When creating the issues from a script, line breaks were written as `\n` and dollar signs as `\$`, so the issue text showed those characters instead of new lines.

**Fix**
- Rewrote each issue body with real line breaks (`printf '%b'`) and removed the stray backslashes.

**Learned**
- Text inside a shell script isn't always passed through the way it looks. Check the result on GitHub after any bulk change.

---

## 2026-11-17: Connecting the AWS CLI (Agent Toolkit for AWS setup)

**Did**
- Chose the AWS region **eu-north-1** (Stockholm) and the CLI profile name `ai`.
- AWS CLI (v2.40.2) and `uv` were already installed, so the install steps were skipped.
- Ran `aws login --region eu-north-1 --profile ai`.

**Mistake / problem**
- `aws login` failed three times with `AccessDeniedException ... CreateOAuth2Token: The refresh token has expired (TOKEN_EXPIRED)`.
- Ruled out: an old saved login (none existed). `--remote` mode failed the same way.

**Cause**
- My PC clock. `w32tm /query /status` showed Windows had **never** synced its time (`Source: Local CMOS Clock`, `not synchronized`) and was about 45 seconds off. The AWS login code is only valid for a very short time, so even a small clock error makes AWS reject it. This is a known problem: [aws/aws-cli#10330](https://github.com/aws/aws-cli/issues/10330).

**Fix**
- Settings → Time & language → Date & time → turn on *Set time automatically* → **Sync now**. Then run `aws login` again. ✅ It worked: `aws sts get-caller-identity --profile ai` now returns my project's account.

**Then: installed the Agent Toolkit for AWS**
- `aws configure agent-toolkit --yes --region us-east-1 --profile ai` installed 25 AWS skills for Claude Code and added the **AWS MCP server** (lets the AI assistant read AWS docs and work with my AWS project). The toolkit service only runs in us-east-1; my own resources still go in eu-north-1.
- Pointed the MCP server at my `ai` profile (`AWS_MCP_PROXY_PROFILES=ai`), otherwise it looks for a `default` profile that doesn't exist.
- Checked it works: `aws agent-toolkit list-available-skills` lists 126 skills.
- Added AWS's rules for the new AWS experience to a new [CLAUDE.md](CLAUDE.md). Notable limit: everything must be created in **eu-north-1**, my project's only Region.

**Learned**
- "Expired" errors in login systems often mean *your clock is wrong*, not that you were too slow. Check time sync first.
- An *MCP server* is a plug-in that gives an AI assistant new tools; *skills* are instruction packs it loads when needed.

---

## 2026-11-17: Project start

**Did**
- Decided the goal: one open-source base model, many private assistants, hosted on AWS, built with Terraform, deployed by CI/CD.
- Checked my PC. Already installed: Git, GitHub CLI, Terraform, AWS CLI, Python 3.11. Missing: Docker, VS Code (not needed yet).
- Created this repo with a README, roadmap and this log.

**Problems found**
- GitHub CLI not logged in, AWS CLI not configured, Git has no name/email set. All fixed in Phase 0.

**Learned**
- A `.gitignore` file stops secrets (passwords, keys, Terraform state) and huge model files from ever being uploaded to GitHub. Added it before the first commit on purpose.
