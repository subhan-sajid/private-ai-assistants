# Build Log

Newest entries at the top. Every entry: what I did, what went wrong, how I fixed it, what I learned.

---

## 2026-11-17: Connecting the AWS CLI (Agent Toolkit for AWS setup)

**Did**
- Chose the AWS region **eu-north-1** (Stockholm) and the CLI profile name `ai`.
- AWS CLI (v2.40.2) and `uv` were already installed, so the install steps were skipped.
- Ran `aws login --region eu-north-1 --profile ai`.

**Mistake / problem**
- `aws login` failed three times with `AccessDeniedException ... CreateOAuth2Token: The refresh token has expired (TOKEN_EXPIRED)`.
- Ruled out: an old saved login (none existed) and my PC clock (only 45 seconds off AWS).

**Fix**
- _In progress._

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
