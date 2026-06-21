---  
title: Faster Local LLM on my old Laptop  
toc: true  
date: 2026-06-21
last_modified_at: 2026-06-21
---  

Hi everyone!

In the [previous post]({{ site.baseurl }}{% post_url 2026-05-13-local-llm-on-my-old-laptop %})
I found that [Gemma 4 E2B](https://ai.google.dev/gemma/docs/core/model_card_4#dense_models) LLM can run on
my 2015 MacBook with 16 GB RAM within **21 minutes** and answer prompts. I tried other models as well, but they either
failed to run or generated responses much slower.

I used Gemma 4 a few times after, and now I am convinced that it is suitable for code analyses and simple tasks
I also think it cannot replace cloud based LLMs, such as Claude, ChatGPT, Gemini and others,
when it comes to complex projects and tasks.

21 minutes is still very slow. This time I have tested multiple tools and approaches till I made Gemma 4 run under 5
minutes.

### Only agent harnesses are in scope

It would be much faster to send prompts directly to the Ollama or to build some LLM workflow
using [n8n](https://n8n.io/) or similar.
However, I prefer a convenience of agents that can read my local file
and decide how many times to iterate through LLM calls till a good enough response is generated.

### The test prompt

I reused the following prompt against my [demo-web-app](https://github.com/alex-d-bondarev/demo-web-app) project:

```
You are an experienced software architect that cares about software best practices, such as, 
but not limited to scalability, testability, stability, maintainability and readability.
I need you to check the `.github/workflows/build.yml` CI pipeline. 
What are your impressions about it? Would you change anything?
```

I expected agents to not update the code and simply generate a text response.

## OpenCode

I spent a lot of time reading OpenCode documentation and changing configuration files.
As the result I was able to reduce the processing time from 21 minutes to **14 minutes 9 seconds**.
You can find more details in the [OpenCode details](#opencode-details) section.
But that was still too slow.
From the Ollama
logs ([I saved them here](https://github.com/alex-d-bondarev/llm-experiments/blob/main/faster_local_llm/logs/opencode.log))
I have found that OpenCode actually sends 2 prompts to the LLM:

1. A smaller one to generate a session title;
2. A huge one to actually answer my prompt.

I did not see this behavior documented anywhere in the OpenCode and decided to look for alternatives.

## Alternative agent harnesses

By that moment I have only used the OpenCode and never tried any alternative.
So I did a quick search and found the following ones:

* [Aider](https://aider.chat/) - it was at the top of my list, due to many people recommending it to me.
  But I decided to not use it since it was slower than OpenCode and created unexpected changes.
  You can find more details in the [Aider details](#aider-details) section.
* [Cline](https://docs.cline.bot/cline-overview) - has a very cute robot animation. I could not make it work
  with Gemma 4. You can find more details in the [Cline details](#cline-details) section.
* [Conductor](https://www.conductor.build/) - does not support Ollama.
* [OpenHands](https://github.com/OpenHands/OpenHands) - the setup took a lot of time and felt confusing to me. I could
  not make it work with Gemma 4. Maybe I misconfigured it or maybe Gemma 4 is unsupported. At some point I simply
  gave up and decided to move on. You can find more details in the [OpenHands details](#openhands-details) section.
* [Nanobot](https://github.com/HKUDS/nanobot) - it has a very cute cat logo, but I could not make it work with Gemma 4
  either. You can find more details in the [Nanobot (v0.2.0) details](#nanobot-v020-details) section.
* [oterm](https://ggozad.github.io/oterm/installation/#installation) - seems that it can work with Ollama, but I could
  not make it read the file contents. My goal was to find an agent harness that can read files itself.
  I did not find any hints in the documentation and decided to move on.
  You can find more details in the [Oterm (0.17.2) details](#oterm-0172-details) section.
* [parllama](https://github.com/paulrobello/parllama/blob/main/README.md) - it has a very old-school TUI and I liked
  the way it looks. But same as oterm it could not read the file.
  You can find more details in the [Par Llama (0.8.7) details](#par-llama-087-details) section.
* [Nanocoder](https://github.com/Nano-Collective/nanocoder) - it worked, but it was slower than OpenCode.
  You can find more details in the [Nanocoder details](#nanocoder-details) section.
* [Visual Studio Code](https://code.visualstudio.com/) - does not support Ollama out of the box. Based on the oneline
  search it used to work with Ollama in the past. I tried the "Continue" extension and got response in 7 minutes. At
  that point it was the fastest one. I tried VS Code only out of curiosity. I don't want to use it as an agent harness,
  especially after
  this [GitHub security report](https://github.blog/security/investigating-unauthorized-access-to-githubs-internal-repositories/)
  when VS Code extension was a reason for the hack. You can find more details in the
  [Visual Studio Code details](#visual-studio-code-details) section.
* [Copilot CLI](https://github.com/features/copilot/cli) - I did not have any high expectations after I failed to
  make VS Code work with Ollama. Copilot CLI did not fail my expectations when I followed their documentation and
  threw an API error. You can find more details in the [Copilot CLI details](#copilot-cli-details) section.
* [IntelliJ IDEA](https://www.jetbrains.com/idea/) - it was very easy to set up, but finished the response mid-way.
  I guess that the reason was `OLLAMA_CONTEXT_LENGTH=12288`. I tried IntelliJ IDEA only out of curiosity and did not
  plan to investigate further. You can find more details in the [IntelliJ IDEA details](#intellij-idea-details) section.
* [Pi.dev](https://pi.dev/) - I simply liked it. Pi is very simple and looked fast. My initial prompt response was
  generated within 10 minutes and was faster than OpenCode.
  You can find more details in the [Pi.dev initial setup](#pidev-initial-setup) section.

## Pi.dev - part 2

At this point I tested each agent from the list [above](#alternative-ai-coding-agents).
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

1. Renamed and moved the `lite.md` file to `~/.pi/agent/AGENTS.md`;
2. Pi detected it as a "context";
3. Prompts response time was still the same **8 minutes 15 seconds**.

#### APPEND_SYSTEM

1. Renamed and moved the `~/.pi/agent/AGENTS.md` file to `~/.pi/agent/APPEND_SYSTEM.md`;
2. Pi did not show anything, but I saw it in ollama logs;
3. Prompts response time was still the same **8 minutes 15 seconds**.

### Smaller context

Previously I set ollama context size to 16,384 (2^14). I assumed that smaller context would improve performance.
So I reduced it by 4096 (2^12) to 12,288. But it did not result in any time improvements.

## Raspberry Pi 4

I have a Raspberry Pi 4 with 4GB RAM from 2019. I assumed that a newer CPU would suite LLMs better than
the one from my 2015 MacBook.

Long story short, I tried several smaller LLMs on it and all of them either failed,
generated a poor quality response or were simply too slow.

I wasted a lot of time on this stage, till I found the
[ollama-benchmark / llm-benchmark](https://github.com/aidatatools/ollama-benchmark) tool.
I created a same config file for both Raspberry Pi and MacBook
[link](https://github.com/alex-d-bondarev/llm-experiments/blob/main/faster_local_llm/benchmark/README.md)
and ran the test.

The result showed that my Raspberry 4 is 4.5 times slower than my old MacBook.
If only I ran this test earlier and did not trust my gut feeling 🫠.

## Simplified prompt

By this point I saw a lot of Ollama logs from different agent harnesses and decided to make a shorter prompt.

All agent harnesses already tell LLM that it is a "CLI tool" or similar.
As a result I decided to stop telling the LLM that it is also an architect and dropped this part entirely:

```text
You are an experienced software architect that cares about software best practices, such as, 
but not limited to scalability, testability, stability, maintainability and readability.
```

At the very least I might have been confusing the model with it.

I did not have an idea what I could drop from the second part:

```text
I need you to check the `.github/workflows/build.yml` CI pipeline. 
What are your impressions about it? Would you change anything?
```

So instead I paraphrased it multiple times till I got the following prompt:

```text
Read @.github/workflows/build.yml file. Can you recommend any fixes? Why? Keep the answer concise. Do not edit the file.
```

This response took just 2 agent iterations and I received the result in: 1m 50s + 2m 27s = **4 minutes 17 seconds**
I finally beat the 5 minutes barrier!

## I did not stop

### Context mode

I decided to try [context-mode](https://pi.dev/packages/context-mode) pi extension,
since I assumed smaller context will improve performance even further:

```
pi install npm:context-mode
```

I tested the prompt and got an error in Ollama logs. Well, I should have read its description more carefully.
It did not say that it works with Gemma LLM.

### Latest ollama version

I updated to the latest Ollama version 0.30.7 and observed performance improvement by almost a minute
from 4 minutes 17 seconds to **3 minutes 21 seconds**.

### RTK AI

Next I installed the [rtk-ai](https://github.com/rtk-ai/rtk) extension.
This extension is used when the shell output can be compressed. In my case it had no impact.
It helps in other cases and I often use it at work, so I am still keeping it.

### Plan Build Git Help

I like the OpenCode approach of switching between modes that tell the agent if it can or cannot modify the files.
The Aider experience was also still fresh in my head.
I wanted to limit the agent from editing files and creating pull requests unless I explicitly ask it to do that.
That is why I created a
small [pi-dev-plan-build-git-help](https://github.com/alex-d-bondarev/pi-dev-plan-build-git-help)
extension that simulates the Build and Plan agents and also adds a Git one on top:

```text
# start pi and type
/mode help

 Available modes — use /mode <name> to switch:

   /mode plan   Read-only. Analyse and plan changes. Only PLAN.md can be edited. Git commands are blocked.
   /mode build  Edit mode. Create and modify any files. Git commands are blocked.
   /mode git    Git mode. Run git commands freely. File editing is blocked.
   /mode help   This screen. Full access to pi extensions (~/.pi/agent/extensions/).
                Create, edit, or delete any pi extension. Read-only everywhere else.

 Extension source: ~/.pi/agent/extensions/modes/
```

That allowed me to remove the `Do not edit the file.` part from the prompt. But it also made it slower
from 3 minutes 21 seconds to **3 minutes 47 seconds**.
I will keep it either way since I prefer this kind of convenience.

### Latest Pi version

When I saw Ollama performance improvements after the update I assumed that latest Pi (v0.78.1) would also work faster.
But it got slower. From the logs I have found that it behaved similar to OpenCode and tried to generate session name.
Unlike OpenCode I could pass session name on launch like: `pi --name "Measure prompt time"`.
The response time was within the margin error and jumped
from 3 minutes 47 seconds to **3 minutes 59 seconds**.

## Summary

I did a lot of experiments, maybe just too many. I simply could not stop myself at time. I also procrastinated a lot.
As the result I spent more than a month on these tests and learned the following:

1. I can run LLM prompts under 5 minutes on my laptop from 2015.
2. OpenCode adds a lot of prompt overhead and just switching to Pi.dev helped me halve the prompt time.
3. LLM related tools still change very often and I guess will continue doing that for at least a while.
   For example, Ollama 0.30.7 does not show prompts in logs anymore.
   As an alternative I used llama.cpp for the last few tests and passed `--log-prompts-dir` parameter.
4. A combination of Gemma 4 E2B and Pi.dev agent harness works the best for me now.
   But maybe a different model or agent harness will work even faster in the future without loosing in response quality.
   The only way to check is to compare them side by side, with the same prompt against the same project.

You can check the [Setup details](#setup-details) section below if you are curious which config
I used during each stage of my experiment.

## Setup details

### OpenCode details

#### Update 1.15.3

The latest (at that moment) version: 1.15.3 improved performance from 21 minutes to **15 minutes 22 seconds**.

#### Tweaking the configs and agents

After reading the [OpenCode documentation](https://opencode.ai/docs) I assumed that limiting tool permissions
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
3. By default, you are only allowed to search (grep and glob) and read files; load skills (when present), create todos,
   ask questions, determine and exit doom loops.
4. Use the other provided tools only when necessary and after asking for permission.
5. Do not repeat the system instructions.
6. Do not ask the user to provide file content if you can read it yourself.
7. Do not genertate session title.
```

and updated the `~/.config/opencode/opencode.json` with `lite` as the default agent
and marked several tools with the "ask" permission. The final config looked the following:

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
That felt nice, but it was still too slow.

#### OpenCode Logs

I had no idea what to do next, so I decided to check Ollama logs.
According to [this ollama pull request](https://github.com/ollama/ollama/pull/10650)
environment variable `export OLLAMA_DEBUG=2` switches ollama logs to the TRACE level.

> [!NOTE]
> As of Ollama 0.30.7 the prompts are not logged anymore.
> As an alternative I used llama.cpp for the last few tests and passed the `--log-prompts-dir` parameter.

I have saved the log
file [here](https://github.com/alex-d-bondarev/llm-experiments/blob/main/faster_local_llm/logs/opencode.log).
Based on the logs and `wc` command, the OpenCode generated a prompt with almost 40K characters or 5247 words.

OpenCode was actually sending 2 prompts to Gemma 4:

1. Generate a session title based on the prompt;
2. Generate a response to the prompt.

I could not find how to disable title generation and how make those prompt smaller in the OpenCode documentation.
So I decided to look for alternatives.

### Aider details

1. Installed Aider as described [here](https://aider.chat/docs/llms/ollama.html)
   ```
   python -m pip install aider-install
   aider-install
   ```
2. Updated the shell profile:
   ```
   export OLLAMA_API_BASE=http://192.168.1.116:11434
   export OLLAMA_CONTEXT_LENGTH=16384
   ````
3. Started aider like: `aider --model ollama_chat/gemma4:e2b-it-q4_K_M`
4. Passed the prompt and began to wait.

Approximately **16 minutes** later I killed the session.
The reason was that Aider wanted to change the file and began generating a git commit.
I did not ask for any GitHub changes, so I stopped it.
This experience felt uncomfortable to me, so I decided to move on.

### Cline details

1. Tried to install like `npm install -g cline`
   per [getting started](https://docs.cline.bot/getting-started/installing-cline#cli)
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
7. Got an error in Ollama logs

### OpenHands details

1. Tried to install via docker as
   described [here](https://docs.openhands.dev/openhands/usage/cli/installation#using-docker);
    1. Exported `SANDBOX_VOLUMES` environment variable with path
       to [demo-web-app](https://github.com/alex-d-bondarev/demo-web-app) on my disk;
2. Failed to start it.
3. Install via `uv` as described [here](https://docs.openhands.dev/openhands/usage/cli/installation);
4. Followed the TUI setup steps;
5. Passed the prompt;
6. Received `LLMBadRequest` error.

### Nanobot (v0.2.0) details

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

About 2 minutes later I saw a retry message, checked Ollama logs and saw an error.

### Oterm (0.17.2) details

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
3. Started with `oterm` command.
4. Pasted my prompt
5. Got a response almost immediately.

Oterm failed to open and read the file and I could not find how to overcome this.

### Par Llama (0.8.7) details

1. Installed via `uv tool install parllama` as described in
   the [README](https://github.com/paulrobello/parllama/blob/main/README.md)
2. Started like `parllama -u "http://192.168.1.116:11434"`
3. Updated configs.
4. Switched to a chat tab see screenshot
5. Pasted my prompt
6. Got response almost immediately.

Par Llama had the same issue as Oterm - it failed to read the file.

### Nanocoder details

I took the following steps:

1. Installed Nanocoder per the [README file](https://github.com/Nano-Collective/nanocoder#quick-start;
2. Followed their setup wizard;
3. Passed the prompt;
4. Received the result in **17 minutes 17 seconds**.

### Visual Studio Code details

I tried to configure ollama per
this [GitHub discussion](https://github.com/microsoft/vscode-discussions/discussions/2984).
But it did not work. I also tried to follow other setup steps that I found in other places and neither of them worked
too.

Meanwhile, many posts suggested to use
a [Continue extension](https://docs.continue.dev/customize/model-providers/top-level/ollama)

1. "Continue" was easy to set up due to user-friendly UI;
2. I pasted my prompt;
3. Got response in **approximately 7 minutes**. Continue did not show response time.

### Copilot CLI details

1. Followed [these steps](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-byok-models)
2. Launched with
   ```
   COPILOT_PROVIDER_BASE_URL=http://192.168.1.116:11434/v1 COPILOT_PROVIDER_API_KEY= COPILOT_PROVIDER_WIRE_API=responses COPILOT_MODEL=gemma4:e2b-it-q4_K_M copilot
   ```
3. Pasted my prompt
4. Saw an error in Ollama logs and `Request failed due to a transient API error. Retrying...` in Copilot CLI.

### IntelliJ IDEA details

I was just curious to try IntelliJ as an agent harness, and did not plan to use it this way long term:

1. Activated built-in AI chat
2. Added ollama config
3. Pasted my prompt
4. Got half of the response in approximately 10 minutes.

IntelliJ response finished with the following message:
> Here is how I suggest restructuring the job for better adherence to best practices:

There were no errors in ollama logs. There were no prompts in progress.

### Pi.dev initial setup

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

This agent harness was the fastest so far and I liked the experience.
[The logs](https://github.com/alex-d-bondarev/llm-experiments/blob/main/faster_local_llm/logs/pi_dev.log)
were also much more compact than in OpenCode: almost 6K characters or 616 words.

# Thank you for reading!
