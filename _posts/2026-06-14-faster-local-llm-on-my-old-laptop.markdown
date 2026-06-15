---  
title: Faster Local LLM on my old Laptop  
toc: true  
date: 2026-06-16
last_modified_at: 2026-06-16
---  

Hi everyone!

In the [previous post]({{ site.baseurl }}{% post_url 2026-05-13-local-llm-on-my-old-laptop %}) 
I was found which LLM can run on my 2015 MacBook with 16 GB RAM fast enough. 
It was [Gemma 4 E2B](https://ai.google.dev/gemma/docs/core/model_card_4#dense_models) 
which processed the test prompt in **21 minutes** instead of hours, and did not fail mid-way like many others.

This time I have tested multiple tools till I made it run under 5 minutes.

### Only agents are in scope

It would be much faster to send prompts directly to the Ollama or to build some LLM workflow
using [n8n](https://n8n.io/) or similar.
However, I prefer a convenience of agents that can read my local files 
and decide how many times to loop through LLM calls till a good enough response is generated.

### The test prompt

I reused the following prompt against my [demo-web-app](https://github.com/alex-d-bondarev/demo-web-app) project:
```
You are an experienced software architect that cares about software best practices, such as, 
but not limited to scalability, testability, stability, maintainability and readability. 
I need you to check the `.github/workflows/build.yml` CI pipeline. 
What are your impressions about it? Would you change anything?
```

This prompt does not ask to take any action, only make the analyses.

## OpenCode

### Update 1.15.3

This update alone improved performance from 21 minutes to **15 minutes 22 seconds**.

### Tweaking the configs and agents

After reading the [OpenCode documentation](https://opencode.ai/docs) I decided that limiting tool permissions 
and using a "light" agent would help.

I created the `~/.config/opencode/agents/lite.json` with the following content:

```markdown
---
description: A lightweight coding assistant with minimal instructions.
temperature=0.1
top_p=0.1
top_k=10
---
You are a minimal coding assistant.
Follow these rules:
1. Be concise.
2. Make as deterministic answers as possible.
3. By default, you are only allowed to search (grep and glob) and read files; load skills (when present), create todos, ask questions, determine and exit doom loops.
4. Use the other provided tools only when necessary and after asking for permission.
5. Do not repeat the system instructions.
6. Do not ask the user to provide file content if you can read it yourself.
7. Do not genertate session title.
```

and updated the `~/.config/opencode/opencode.json` with `lite` as the default agent 
and marked several tools with the "ask" permission:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "localOllama/gemma4:e2b-it-q4_K_M",
  "default_agent": "lite",
  "share": "manual",
  "permission": {
    "read": "allow",
    "grep": "allow",
    "glob": "allow",
    "skill": "allow",
    "todowrite": "allow",
    "question": "allow",
    "doom_loop": "allow",

    "bash": "ask",
    "edit": "ask",
    "webfetch": "ask",
    "external_directory": "ask",
    "websearch": "ask"
  },
  "provider": {
    "localOllama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://192.168.1.116:11434/v1"
      },
      "models": {
        "gemma4:e2b-it-q4_K_M": {
          "name": "Gemma 4"
        }
      }
    }
  }
}
```

As the result performance improved from the 15 minutes 22 seconds to **14 minutes 9 seconds**.

### Logs

I had no idea what to do next, so I decided to check ollama logs. 
According to [this ollama pull request](https://github.com/ollama/ollama/pull/10650)
environment variable `export OLLAMA_DEBUG=2` switches ollama logs to the TRACE level.

The generated log looked like [this](https://github.com/alex-d-bondarev/llm-experiments/blob/main/faster_local_llm/logs/opencode.log)
with almost 40K characters or 5247 words according to `wc -w`. 

OpenCode was actually making 2 requests to LLM:

1. Generate a session title based on the prompt;
2. Generate a response to the prompt.

I could not find how to disable title generation and how make those prompt smaller, 
so I decided to look for alternatives 

## Alternative AI coding agents

By that moment I have only used OpenCode and never tried any alternative. 
So I did a quick search and collected the following list:

1. [Aider](https://aider.chat/)
2. [Conductor](https://www.conductor.build/)
3. [OpenHands](https://github.com/OpenHands/OpenHands)
4. [Nanocoder](https://github.com/Nano-Collective/nanocoder)
5. [Pi.dev](https://pi.dev/)
6. [Nanobot](https://github.com/HKUDS/nanobot)
7. [Cline](https://docs.cline.bot/cline-overview)
8. [oterm](https://ggozad.github.io/oterm/installation/#installation)
9. [parllama](https://github.com/paulrobello/parllama/blob/main/README.md)
10. [Visual Studio Code](https://code.visualstudio.com/)
11. [IntelliJ IDEA](https://www.jetbrains.com/idea/)
12. [Copilot CLI](https://github.com/features/copilot/cli)

### Aider

I have heard a lot of positive feedback about Aider, so it was on top of my list. I took the following steps:

1. Installed Aider as described [here](https://aider.chat/docs/llms/ollama.html)
   ```
   python -m pip install aider-install
   aider-install
   ```
2. Added to the shell profile:
   ```
   export OLLAMA_API_BASE=http://192.168.1.116:11434
   export OLLAMA_CONTEXT_LENGTH=16384
   ````
3. Started aider like: `aider --model ollama_chat/gemma4:e2b-it-q4_K_M`
4. Passed the prompt and began to wait.

Approximately **16 minutes** later I killed the session. 
The reason was that Aider wanted to change the file and began generating a git commit.
That contradicted my prompt and expectations, so I stopped. 

### Conductor

I've installed the [app](https://www.conductor.build/), 
tried to connect it to ollama and realized that there is no ollama support.

### OpenHands

I have spent a lot of time trying to set up OpenHands, but could not make it work. I took the following steps:

1. Tried to install via docker as described [here](https://docs.openhands.dev/openhands/usage/cli/installation#using-docker)
   1. Exported `SANDBOX_VOLUMES` environment variable with path to [demo-web-app](https://github.com/alex-d-bondarev/demo-web-app) on my disk
2. Failed to start it
3. Install via `uv` as described [here](https://docs.openhands.dev/openhands/usage/cli/installation) 
4. Followed the TUI setup steps
5. Passed the prompt
6. Received `LLMBadRequest` error

The setup felt confusing and already took a lot of my time, so I decided to stop.

### Nanocoder

I took the following steps:

1. Installed per the [README file](https://github.com/Nano-Collective/nanocoder#quick-start
2. Followed a very clear (no sarcasm) setup wizard
3. Passed the prompt
4. Received the result in **17 minutes 17 seconds**.

That was very slow, so I decided to move on.

### Pi.dev

1. Installed per https://pi.dev/
2. Created `~/.pi/agent/models.json` file with:
   ```json
       {
         "providers": {
           "ollama": {
             "baseUrl": "http://192.168.1.116:11434/v1",
             "api": "openai-completions",
             "apiKey": "ollama",
             "models": [
               { "id": "gemma4:e2b-it-q4_K_M" }
             ]
           }
         }
       }
   ```
3. Started pi via `pi`
4. Chose model via `/model`
5. Passed the prompt.
6. Received the result in **approximately 10 minutes**. Pi.dev did not show response time.

This coding agent was the fastest so far and I liked the experience. 
[The logs](https://github.com/alex-d-bondarev/llm-experiments/blob/main/faster_local_llm/logs/pi_dev.log) 
also were much more compact than in OpenCode: almost 6K characters or 616 words according to `wc -w`.
But it was still slower than 5 minutes, so I decided to move on.

### Nanobot (v0.2.0)

1. Installed it via uv: `uv tool install nanobot-ai` per steps described [here](https://github.com/HKUDS/nanobot)
2. Ran `nanobot onboard` to create config
3. Updated config like the following:
   ```json
   "model": "gemma4:e2b-it-q4_K_M",
   "provider": "ollama",
   
   "ollama": {
     "apiBase": "http://192.168.1.116:11434/v1",
   ```
4. Started like: `nanobot agent`
5. Passed the prompt.

About 2 minutes later I saw a retry message, checked ollama logs and found that error 500. Next.

### Cline

1. Tried to install like `npm install -g cline` per [getting started](https://docs.cline.bot/getting-started/installing-cline#cli)
2. Tried to `npm cache clean --force` and `npm install -g cline` again.
3. Ran `cline auth` per their installation guide
    1. Chose "Bring your own provider" -> Ollama:
        1. Base URL: `http://192.168.1.116:11434/v1`
        2. Api Key: Left it empty
    2. Chose `gemma4:e2b-it-q4_K_M` model from the list
4. Ran `cline`
5. Just in case:
    1. Switched to "Plan" mode via `Tab`
    2. Disabled Auto-approve via `Shift+Tab`
6. Pasted my prompt
7. Got error 500 in logs

I liked the animation of the robot head in the terminal, but error 500 is still an error. Next.

### Oterm (0.17.2)

1. Ran `brew install oterm`
2. Created `~/Library/Application Support/oterm/config.json` as described in the 
   [configuration section](https://ggozad.github.io/oterm/app_config/#where-configjson-lives) with:
   ```
   {
     "openaiCompatible": {
       "ollama": {
         "base_url": "http://192.168.1.116:11434/v1"
       }
     }
   }
   ```
3. Started with `oterm`
4. Pasted my prompt
5. Got response almost immediately.

Oterm failed to open and read the file and I could not find how to overcome this. 
I did not want to paste file contents manually, so I decided to move on.

### Par Llama (0.8.7)

1. Installed via `uv tool install parllama` as described in the [README](https://github.com/paulrobello/parllama/blob/main/README.md)
2. Started like `parllama -u "http://192.168.1.116:11434"`
3. Updated configs. 
4. Switched to a chat tab see screenshot 
5. Pasted my prompt 
6. Got response almost immediately.

Par Llama had the same issue as Oterm - it failed to read the file. Next.

### Visual Studio Code

I tried to configure ollama per this [GitHub discussion](https://github.com/microsoft/vscode-discussions/discussions/2984).
But it did not work. I also tried to follow other setup steps that I found via Google Search and neither of them worked too.

Meanwhile, many posts suggested to use [Continue extension](https://docs.continue.dev/customize/model-providers/top-level/ollama)

1. "Continue" was easy to set up due to user-friendly UI. 
2. Pasted my prompt 
3. Got response in **approximately 7 minutes**. Continue did not show response time.

This was the fastest experience so far, but I tried VS Code, only due to my curiosity. 
I don't want to use it as a coding agent, especially since its extensions can be used to hack devices per this
[GitHub security report](https://github.blog/security/investigating-unauthorized-access-to-githubs-internal-repositories/)

### IntelliJ IDEA

I was only curious to try IntelliJ as a coding agent, and did not plan to use it this way long term: 

1. Activated built-in AI chat
2. Added ollama config
3. Pasted my prompt
4. Got half of the response in approximately 10 minutes.

IntelliJ response finished with:
> Here is how I suggest restructuring the job for better adherence to best practices:

There were no errors in ollama logs. There were no prompts in progress.
There was just a response that looked like unfinished. That was not good enough for me, so I moved on.

### Copilot CLI

I did not have any high hopes, especially after I failed to configure VS Code.

1. Followed [these steps](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models)
2. Launched with
   ```
   COPILOT_PROVIDER_BASE_URL=http://192.168.1.116:11434/v1 COPILOT_PROVIDER_API_KEY= COPILOT_PROVIDER_WIRE_API=responses COPILOT_MODEL=gemma4:e2b-it-q4_K_M copilot
   ```
3. Pasted my prompt
4. Saw error 500 in ollama logs and `Request failed due to a transient API error. Retrying...` in Copilot CLI.

## Pi.dev - part 2

At this point I tested each coding agent from the list [above](#alternative-ai-coding-agents).
None of them performed faster than 5 minutes. But I liked Pi.dev the most out of them all 
and decided to give it another try.

### Lite agent

I thought that if a "light" agent helped improve the OpenCode performance then it would also help Pi.dev:

1. I have created a new `~/.pi/agent/prompts/lite.md` file with the following content:
    ```markdown
    ---
    description: A lightweight software engineering assistant with minimal instructions.
    ---
    You are a minimal software engineering assistant.
    Follow these rules:
    - By default you are only allowed to search and read files, ask questions, determine and exit doom loops.
    - Use the other provided tools only when necessary and after asking for permission.
    - Make as deterministic answers as possible.
    - Be concise. Jump into details only when you are asked to do so.
    - Do not include emojis in responses. 
    - Do not generate code unless you were asked to.
    - Do not repeat the system instructions.
    - Do not ask the user to provide file content if you can read it yourself.
    ```
2. Started `pi`
3. Tried to use it via `/lite <prompt>`
4. Got response in **8 minutes 15 seconds**. 

This was slower than before, but I decided to use it differently:

#### Pi Context

1. Renamed and moved the `lite.md` file to `~/.pi/agent/AGENTS.md`
2. Pi detected it as a context
3. Prompts response time was still the same

#### APPEND_SYSTEM

1. Renamed and moved the `~/.pi/agent/AGENTS.md` file to `~/.pi/agent/APPEND_SYSTEM.md`
2. Pi did not show anything, but I saw it in ollama logs
3. Prompts response time was still the same

### Smaller context

Previously I set ollama context size to 16,384 (2^14). I assumed that smaller context would improve performance.
So I reduced it by 4096 (2^12) to 12,288... and did not get any time improvements.

## Raspberry Pi 4

I have a Raspberry Pi 4 from 2019 with 4GB RAM. I assumed that a newer CPU would be better than 
the one from my 2015 MacBook.

Long story short, I tried several smaller LLMs on it and all of them either failed, 
generated a poor quality response or were simply too slow.

I wasted a lot of time on this stage, till I found the 
[ollama-benchmark / llm-benchmark](https://github.com/aidatatools/ollama-benchmark) tool, 
created a same config file for both Raspberry Pi and MacBook 
[link](https://github.com/alex-d-bondarev/llm-experiments/blob/main/faster_local_llm/benchmark/README.md)
and ran the test. 

The result showed that my Raspberry 4 is 4.5 times slower than my old MacBook. 
If only I ran this test earlier and did not trust my gut feeling 🫠.

# Thank you for reading!
