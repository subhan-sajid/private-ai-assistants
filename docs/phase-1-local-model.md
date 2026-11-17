# Phase 1: Run the model on your own PC

**Goal:** download an open-source AI model, chat with it on your own PC, talk to it the way an app would, and make your first custom assistant.

**Time:** about 1 hour (most of it is waiting for a ~5 GB download). **Cost:** $0. Nothing in this phase touches AWS.

**Why start on your PC and not in the cloud?** Everything you learn here (models, prompts, the API) works the same on AWS later. On your PC, mistakes are free. In the cloud, a GPU costs about $1 an hour.

---

## First, what is a model?

Think of the model as a very large file of numbers. You give it some text, and it works out what word most likely comes next, again and again, until it has written an answer.

| Term | Meaning | Our model |
|---|---|---|
| **Parameters** | How many numbers are in the file. More usually means smarter, but slower and bigger. | 8 billion ("8B") |
| **Quantisation** | Storing each number with fewer digits so the file is smaller and fits on your GPU. Very little loss in quality. | 4-bit, so ~5 GB instead of ~16 GB |
| **VRAM** | The memory on your graphics card. The model must fit in it to run fast. | RTX 3060 = 12 GB ✅ |
| **Context window** | How much text (your chat so far plus the answer) the model can "see" at once. | Thousands of words |
| **Tokens** | Models read word pieces, not whole words. "Unbelievable" might be 3 tokens. Speed is measured in tokens per second. | |

**Ollama** is a free program that downloads models and runs them on your GPU, so you don't have to set any of that up yourself. It's the easiest way in.

**Which model?** We'll use **Qwen3 8B** (by Alibaba, open-source under the Apache 2.0 licence, which allows personal and commercial use and changes). It's strong for its size, and later we can fine-tune it. Other good choices are listed at <https://ollama.com/library>. You can swap models any time.

---

## Step 1.1: Install Ollama

In PowerShell:

```powershell
winget install Ollama.Ollama
```

`winget` is Windows' built-in app installer. It downloads Ollama from the official source. If Windows asks *"Do you want to allow this app to make changes?"*, click **Yes**.

**Close PowerShell and open a new one** (new programs only show up in new terminal windows), then check:

```powershell
ollama --version
```

You should see a version number. Ollama now runs quietly in the background (a llama icon near the clock).

---

## Step 1.2: Download and chat with the model

```powershell
ollama run qwen3:8b
```

The first time, this downloads ~5 GB, so wait for the progress bars. Then you get a `>>>` prompt. Type a message and press Enter:

```
>>> Explain what a GPU is to a 10 year old.
```

Things to notice:
- You may see a `Thinking...` section first. Qwen3 "thinks out loud" before answering on harder questions. Type `/no_think` at the start of a message to skip it.
- Typing `/?` shows the commands, and `/bye` exits.

Try a few prompts: ask it to write a poem, explain some code, summarise a paragraph. Try a fact it probably doesn't know (something about you). It will either say it doesn't know or **make something up**. That's called *hallucination*, and Phase 3 (feeding it your own documents) is how we fix it.

---

## Step 1.3: Check it's really using your GPU

Open a **second** PowerShell window while a chat is running:

```powershell
ollama ps
```

The `PROCESSOR` column should say `100% GPU`. If it says CPU, it's running on your processor instead and will be ~10× slower, so tell Claude.

Also try:

```powershell
ollama list
```

This lists every model you've downloaded and its size.

---

## Step 1.4: Talk to it the way an app would (the API)

Chatting in the terminal is for humans. Apps (like the one we'll build in Phase 2) talk to Ollama over **HTTP** at `http://localhost:11434`. That's the same way your browser talks to websites, except `localhost` means "this PC".

```powershell
$body = @{
  model    = "qwen3:8b"
  messages = @(
    @{ role = "system"; content = "You are a pirate. Answer in one sentence." },
    @{ role = "user";   content = "What is the cloud?" }
  )
  stream = $false
} | ConvertTo-Json -Depth 5

(Invoke-RestMethod -Uri http://localhost:11434/api/chat -Method Post -Body $body -ContentType "application/json").message.content
```

What's happening:
- `messages` is the conversation. Each message has a **role**:
  - `system` sets hidden rules and personality. **This is the first way to personalise an assistant.**
  - `user` is what the person typed.
  - `assistant` is what the model said earlier (none yet).
- `stream = $false` means "send the whole answer at once" instead of word by word.
- The answer comes back as **JSON** (structured text), and `.message.content` pulls out just the reply.

Change the `system` text to something else (a strict teacher, a football commentator) and run it again. Same model, a completely different personality. **That's the core idea of this whole project.**

---

## Step 1.5: Make your first personal assistant

Ollama can save a model plus a system prompt as a new named model, using a small text file called a **Modelfile**. Your repo already has one: [`assistants/study-buddy/Modelfile`](../assistants/study-buddy/Modelfile).

```powershell
cd $HOME\Documents\ai
ollama create study-buddy -f assistants\study-buddy\Modelfile
ollama run study-buddy
```

Ask it something like *"Help me understand what Terraform is"* and see how differently it behaves from plain `qwen3:8b`.

`ollama list` now shows `study-buddy`. It took almost no extra disk space, because it **reuses the same base model** and only adds the instructions. One base model, many assistants.

**Your turn:** copy the `study-buddy` folder, rename it (e.g. `fitness-coach`), edit the `SYSTEM` text in its Modelfile, and `ollama create` it. Make it your own.

---

## Phase 1 done when…

- [ ] `ollama --version` works
- [ ] You chatted with `qwen3:8b`
- [ ] `ollama ps` shows `100% GPU`
- [ ] The API call in Step 1.4 returned an answer
- [ ] `study-buddy` works, plus one assistant you designed yourself

Tell Claude when you're done (and paste anything that went wrong). Claude will log it, commit your new assistant to GitHub, and start Phase 2: your own Python app that serves several assistants at once.

## Common problems

| Problem | Fix |
|---|---|
| `ollama` is not recognised | Close and reopen PowerShell. Still broken? Restart the PC. |
| Download very slow or stuck | Press `Ctrl+C`, then run the same command again. It resumes where it stopped. |
| Answers very slow, `ollama ps` says CPU | Update your NVIDIA driver (GeForce Experience or nvidia.com), restart, and try again. |
| `Invoke-RestMethod`: unable to connect | Ollama isn't running. Open it from the Start menu, then retry. |
