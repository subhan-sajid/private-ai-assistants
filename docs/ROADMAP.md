# Roadmap: the big picture, in plain English

Read this once before starting. You don't need to understand every word yet. Each phase has its own detailed lesson when we get there.

## The words you'll keep hearing

| Word | What it actually means |
|---|---|
| **Model** | A big file of numbers ("weights") that predicts the next word. "Open-source model" = you're allowed to download that file and run it yourself. |
| **Base model** | The model as downloaded, before you change it. |
| **Inference** | Running the model to get answers. This is what costs money every day. |
| **Fine-tuning / training** | Changing the model's numbers by showing it examples. Costs money once, while it trains. |
| **LoRA adapter** | A small add-on file (tens of MB, not GB) that changes a model's behaviour. This is how one base model becomes many assistants: one base, many small adapters. |
| **RAG** | "Retrieval-augmented generation": before answering, look up relevant bits of *your* documents and paste them into the question. Teaches the assistant facts without training. |
| **System prompt** | Hidden instructions given to the model before every chat ("You are Sam's study helper, be brief..."). The cheapest kind of personalisation. |
| **Cloud / AWS** | Renting someone else's computers by the hour. AWS = Amazon Web Services. |
| **GPU** | The kind of chip that runs AI models fast. GPU servers are the expensive part. |
| **Terraform** | You write a text file describing the servers you want; Terraform creates them. Delete the file's contents and it deletes them. Means your whole setup is rebuildable and lives on GitHub. |
| **CI/CD** | "Continuous integration / continuous deployment". Every time you push code to GitHub, robots (GitHub Actions) test it and deploy it for you. |
| **Cognito** | AWS's login service. Handles usernames, passwords, and "groups" (e.g. people in group `family` may use the `family-assistant`). |

## How you'll personalise each assistant (cheapest first)

1. **System prompt**: personality, tone, rules. Free, instant. Do this first, always.
2. **RAG (documents)**: give it knowledge (notes, PDFs, manuals). Cheap, easy to update.
3. **LoRA fine-tune**: teach it a *style* or a *task format* it keeps getting wrong. Costs a few dollars of GPU time per training run.

Most "personal assistants" only need 1 + 2. We'll still do 3 so you learn it.

## The phases

| # | Phase | You'll learn | Rough cost |
|---|---|---|---|
| 0 | Accounts, tools, safety | Git, GitHub, AWS account security, billing alarms | $0 |
| 1 | Run the model on your PC | What a model is, how to chat with one locally (Ollama) | $0 |
| 2 | Tiny API with multiple assistants | Python, FastAPI, how apps talk to models | $0 |
| 3 | Personalise | System prompts, RAG, a LoRA fine-tune on a rented GPU | ~$1–5 per training run |
| 4 | Logins and access | Cognito user pools, groups, tokens | ~$0 (free tier) |
| 5 | Terraform | Infrastructure as code, state, plan/apply/destroy | $0 until you apply |
| 6 | Deploy to AWS | Docker, ECR, EC2 GPU servers, networking | **GPU ≈ $0.80–1.20/hour while running** |
| 7 | CI/CD | GitHub Actions, tests, auto-deploy, secret-free AWS access (OIDC) | $0 (free minutes) |
| 8 | Web app for every device | A simple web chat that installs on a phone like an app | ~$0 |
| 9 | Monitoring and cost control | CloudWatch logs, alarms, auto-shutdown when idle | ~$0–2/month |

### ⚠️ The one money warning that matters

A GPU server left running 24/7 costs roughly **$600–900 a month**. We'll protect against this three ways: a billing alarm (Phase 0), only running the GPU when you're using it (Phase 6), and auto-shutdown when idle (Phase 9). Always run `terraform destroy` when you finish a session until Phase 9 is done.
