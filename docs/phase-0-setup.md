# Phase 0: Accounts, tools and safety

**Goal:** by the end, your PC can talk to GitHub and AWS, and AWS will email you before you spend real money.

**Time:** about 45 minutes. **Cost:** $0.

Your PC already has Git, GitHub CLI (`gh`), Terraform, the AWS CLI and Python. 

## How to run the commands in this guide

Open **PowerShell**: press the Windows key, type `PowerShell`, press Enter. A blue/black window appears. That's a *terminal*: you type a command, press Enter, it runs.

Move into the project folder first, every time you open a new terminal:

```powershell
cd $HOME\Documents\ai
```

`cd` means "change directory" (go into this folder).

---

## Step 0.1: Tell Git who you are

Git stamps your name on every saved change ("commit"). Use the same email as your GitHub account.

```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

**Check it worked:** `git config --global user.name` prints your name back.

> 💡 Want your real email hidden on GitHub? Go to github.com → your photo → Settings → Emails → tick *Keep my email addresses private*. GitHub shows you an address like `12345+username@users.noreply.github.com`. Use that one in the command above instead.

---

## Step 0.2: Log the GitHub CLI into your account

```powershell
gh auth login --scopes project
```

It asks questions. Use the arrow keys and Enter:

1. *Where do you use GitHub?* → **GitHub.com**
2. *Preferred protocol?* → **HTTPS**
3. *Authenticate Git with your GitHub credentials?* → **Yes**
4. *How would you like to authenticate?* → **Login with a web browser**
5. It shows an 8-character code. Copy it, press Enter, your browser opens, paste the code, click **Authorize**.

`--scopes project` gives permission to manage GitHub *Projects* (the progress board), which we need.

**Check it worked:** `gh auth status` says `Logged in to github.com account <you>`.

✅ **When you reach here, tell Claude "GitHub is logged in"**: Claude will create your repository, upload this project and build the progress board.

---

## Step 0.3: Create an AWS account

1. Go to <https://aws.amazon.com> → **Create an AWS account**.
2. You need an email, a card (AWS checks it's real; we'll keep spending near $0), and a phone.
3. Choose the **Basic support – Free** plan.

The email + password you just made is the **root user**. Root can do *anything*, including closing the account. We'll lock it away and never use it day to day.

---

## Step 0.4: Protect the root user with MFA

MFA = a second step at login (a code on your phone). Without it, a leaked password = someone running crypto miners on your card.

1. Install an authenticator app on your phone (Google Authenticator, Microsoft Authenticator or Authy).
2. In the AWS console, click your account name (top right) → **Security credentials**.
3. **Assign MFA device** → name it `root-phone` → **Authenticator app** → scan the QR code → type two codes in a row → **Add MFA**.

---

## Step 0.5: Set a billing alarm (do NOT skip)

1. Top search bar → type **Budgets** → open **AWS Budgets** → **Create budget**.
2. **Use a template** → **Monthly cost budget**.
3. Budget amount: **10** (USD). Email: yours.
4. **Create budget**.

AWS now emails you when you hit 85% and 100% of $10. In Phase 5 we'll recreate this alarm in Terraform so it's in code.

---

> **Update 2026-11-17:** my account uses the *new AWS experience* (signed up with a project), so I skipped Steps 0.6 and 0.7 and used the simpler `aws login` instead:
>
> ```powershell
> aws configure set region eu-north-1 --profile ai
> aws login --region eu-north-1 --profile ai
> ```
>
> The browser opens, you sign in to your project, and the CLI gets credentials for 12 hours (renewable for 90 days without signing in again). My region is **eu-north-1** (Stockholm), not `us-east-1`. The steps below are kept for anyone on the *advanced* AWS experience.

## Step 0.6: Make a day-to-day user (IAM Identity Center)

Instead of root, you'll log in as a normal admin user. Identity Center gives your PC *temporary* keys that expire, so there are no permanent passwords sitting in files.

1. Top search bar → **IAM Identity Center** → **Enable**. If asked, choose **Enable with AWS Organizations**.
2. Left menu → **Users** → **Add user**. Username e.g. `yourname-admin`, your email, fill in names → **Next** → **Next** → **Add user**. Check your email and accept the invite, set a password, set up MFA when it asks.
3. Left menu → **Permission sets** → **Create permission set** → **Predefined** → **AdministratorAccess** → **Next** → leave session duration at 1 hour → **Next** → **Create**.
4. Left menu → **AWS accounts** → tick your account → **Assign users or groups** → **Users** tab → tick your user → **Next** → tick **AdministratorAccess** → **Next** → **Submit**.
5. Left menu → **Dashboard** → copy the **AWS access portal URL** (looks like `https://d-xxxxxxxxxx.awsapps.com/start`). Note the **Region** shown at the top right of the console too (e.g. `us-east-1`).

---

## Step 0.7: Connect the AWS CLI to that user

```powershell
aws configure sso
```

Answer:

| Question | Answer |
|---|---|
| SSO session name | `ai` |
| SSO start URL | the portal URL from 0.6 step 5 |
| SSO region | the region from 0.6 step 5 |
| SSO registration scopes | just press Enter |

Your browser opens: log in as your new user and click **Allow**. Back in the terminal:

| Question | Answer |
|---|---|
| CLI default client Region | `us-east-1` (cheapest and has the most GPUs; fine to keep) |
| CLI default output format | `json` |
| Profile name | `ai` |

**Check it worked:**

```powershell
aws sts get-caller-identity --profile ai
```

You should see your 12-digit account number and an ARN containing `AdministratorAccess`.

Keys expire after about an hour. When commands start saying the token expired, run:

```powershell
aws sso login --profile ai
```

---

## Phase 0 done when…

- [ ] `git config --global user.name` shows your name
- [ ] `gh auth status` shows you're logged in
- [ ] Root user has MFA
- [ ] $10 budget alarm exists
- [ ] `aws sts get-caller-identity --profile ai` works

Then tell Claude, and we'll log it in [LOG.md](../LOG.md) and start Phase 1 (running the model on your own PC).

## If something goes wrong

Copy the **exact** error text and paste it to Claude. Errors feel scary but they're just the computer telling you what it needs. Every error you hit goes in the log as a "mistake + fix", and that's the most valuable part of this repo for anyone reading it.
