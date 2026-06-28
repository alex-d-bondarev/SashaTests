---
title: Faster Local LLM on my old Laptop
toc: true
date: 2026-06-21
last_modified_at: 2026-06-21
---

Hi everyone!

I have a video version in case you prefer that over reading:

I will add a link soon....

In the [previous post]({{ site.baseurl }}{% post_url 2026-05-13-local-llm-on-my-old-laptop %})
I found that [Gemma 4 E2B](https://ai.google.dev/gemma/docs/core/model_card_4#dense_models) LLM can run on my 2015
MacBook with 16 GB RAM within **21 minutes**
and produce good enough results. I tried other models as well, but they either
failed to run or generated responses much slower.

The 21 minutes result was still too slow. So this time I focused on improving the timing by
testing multiple agent harnesses with a goal to get response in 5 minutes or less.

### Only agent harnesses are in scope

It would be much faster to send prompts directly to the Ollama or to build some LLM workflow
using [n8n](https://n8n.io/) or similar.
However, I prefer a convenience of agent harness that can read my local files
and decide how many times to iterate through LLM calls till a good enough response is generated.

### The test prompt

I reused the following prompt against my [demo-web-app](https://github.com/alex-d-bondarev/demo-web-app) project:

```
You are an experienced software architect that cares about software best practices, such as, 
but not limited to scalability, testability, stability, maintainability and readability.
I need you to check the `.github/workflows/build.yml` CI pipeline. 
What are your impressions about it? Would you change anything?
```

My expectation was a text response and no code changes.

## OpenCode

I went through OpenCode documentation and experimented with configuration files, and found that the
[latest OpenCode with a few disabled tools and a more concise agent](#opencode-details)
can improve response time from 21 minutes to **14 minutes 9 seconds**.

But that was not
enough. [Ollama logs](https://github.com/alex-d-bondarev/llm-experiments/blob/main/faster_local_llm/logs/opencode.log)
showed that OpenCode actually sends 2 prompts to the LLM:

1. A smaller one to generate a session title;
2. A huge one to actually answer my prompt.

I did not see this OpenCode behavior documented anywhere and decided to look for alternatives.

## Alternative agent harnesses

By that time I have only used the OpenCode and never tried any alternative agent harness.
After a quick search I came up with the following list:

* [Aider](https://aider.chat/) - many people recommended it to me. But I decided to not use it since it was slower
  than OpenCode and created unexpected changes. Details in the [Aider details](#aider-details).
* [Cline](https://docs.cline.bot/cline-overview) - has a very cute robot animation. But I could not make it work with
  Gemma 4. Details in the [Cline details](#cline-details).
* [Conductor](https://www.conductor.build/) - does not support Ollama.
* [OpenHands](https://github.com/OpenHands/OpenHands) - the setup took a lot of time and felt confusing to me. I could
  not make it work with Gemma 4. Maybe I misconfigured it or maybe Gemma 4 is unsupported.
  Details in the [OpenHands details](#openhands-details).
* [Nanobot](https://github.com/HKUDS/nanobot) - has a very cute cat logo, but I could not make it work with Gemma 4
  either. Details in the [Nanobot (v0.2.0) details](#nanobot-v020-details).
* [oterm](https://ggozad.github.io/oterm/installation/#installation) - supports Ollama, but did not read the file
  contents. I was looking for an agent harness that can read files, but I did not find in the oterm documentation how to
  fix that. Details in the [Oterm (0.17.2) details](#oterm-0172-details).
* [Par llama](https://github.com/paulrobello/parllama/blob/main/README.md) - has a very old-school TUI and I liked the
  way it looks. But it also could not read the file contents. Details in
  the [Par Llama (0.8.7) details](#par-llama-087-details).
* [Nanocoder](https://github.com/Nano-Collective/nanocoder) - worked, but it was slower than the OpenCode. Details in
  the [Nanocoder details](#nanocoder-details).
* [Visual Studio Code](https://code.visualstudio.com/) - The version I used (1.121.0) does not support Ollama out of the
  box. I tried the [Continue extension](https://docs.continue.dev/customize/model-providers/top-level/ollama) and got
  response in **7 minutes**. I was simply curious and do not plan to use it especially after
  this [GitHub security report](https://github.blog/security/investigating-unauthorized-access-to-githubs-internal-repositories/).
  Details in the [Visual Studio Code details](#visual-studio-code-details).
* [Copilot CLI](https://github.com/features/copilot/cli) - I followed their documentation and faced an API error.
  Details in the [Copilot CLI details](#copilot-cli-details).
* [IntelliJ IDEA](https://www.jetbrains.com/idea/) - was very easy to set up, but finished the response mid-way. Since
  it ran out of my limited conext `OLLAMA_CONTEXT_LENGTH=12288`. I tried IntelliJ IDEA only out of curiosity and did not
  plan to push it further. Details in the [IntelliJ IDEA details](#intellij-idea-details).
* [Pi.dev](https://pi.dev/) - I simply liked it. The Pi looked very simple and fast. My initial prompt response was
  generated faster than OpenCode: within **8 minutes 15 seconds**. Details in the [Pi initial setup](#pi-initial-setup).

## Pi.dev

I decided to read Pi.dev documentation and test if I could speed it up any further. I wrongly assumed that a "light"
agent would improve performance same as for the OpenCode and unsuccessfully
tried [a light agent](#pi-lite-agent), [a lite context](#pi-lite-context), and
to [append system markdown](#pi-append_system). Smaller [Ollama context size](#smaller-ollama-context) did not help
either. I tested [my Raspberry Pi](#raspberry-pi-4) as an alternative to the old MacBook and learned that it actually is
4.5 times slower than my laptop despite being 4 years newer.

Finally, I decided to [simplify my prompt](#simplified-prompt) to

```text
Read @.github/workflows/build.yml file. Can you recommend any fixes? Why? Keep the answer concise. Do not edit the file.
```

which resulted in a similar response quality that took just **4 minutes 17 seconds**.

## I did not stop

Did I say "finally"? Well, I did not stop. I have also tested the [pi context-mode extension](#context-mode)
that did not work with Gemma 4, updated to the [latest (0.30.7) Ollama version](#latest-ollama-version),
installed [rtk extension](#rtk-ai), created my custom [Plan Build Git Help](#plan-build-git-help) extension that
mimics Plan and Build from the OpenCode, and updated [pi to the latest version](#latest-pi-version).

The final-final prompt was the following:

```text
Read @.github/workflows/build.yml file. Can you recommend any fixes? Why? Keep the answer concise.
```

and took just **3 minutes 59 seconds** to generate a response

## Summary

I ran a lot of experiments, probably too many. I simply could not stop myself at times.
But I also procrastinated a lot in between them. As the result I spent more than a month on these tests.

My 2015 Laptop can process LLM prompts under 5 minutes, but I will not use it as an LLM server. Instead, I have set up
my M1 MacBook to run Ollama with Gemma 4 E2B and context size 128K. I have also installed Pi.dev
with [rtk-ai](https://github.com/rtk-ai/rtk)
and [pi-dev-plan-build-git-help](https://github.com/alex-d-bondarev/pi-dev-plan-build-git-help), and now get LLM
responses in 30-90 seconds.

Thank you for reading!

## Technical details

I tried to keep the text above short and readable and put all the technical details below:

### OpenCode details

#### Update 1.15.3

The latest (at that moment) OpenCode version 1.15.3 improved performance from 21 minutes to **15 minutes 22 seconds**.

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
According to [this Ollama pull request](https://github.com/ollama/ollama/pull/10650)
environment variable `export OLLAMA_DEBUG=2` used to switch Ollama logs to the TRACE level.

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

I tried to configure Ollama per
this [GitHub discussion](https://github.com/microsoft/vscode-discussions/discussions/2984).
But it did not work. I also tried to follow other setup steps that I found in other places and neither of them worked
too.

Meanwhile, many posts suggested to use
a [Continue extension](https://docs.continue.dev/customize/model-providers/top-level/ollama):

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
2. Added the Ollama config
3. Pasted my prompt
4. Got half of the response in approximately 10 minutes.

IntelliJ response finished with the following message:
> Here is how I suggest restructuring the job for better adherence to best practices:

There were no errors in the Ollama logs. There were no prompts in progress.

### Pi.dev details

#### Pi initial setup

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

#### Pi Lite agent

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
2. Started `pi`;
3. Tried to use it via `/lite <prompt>`;
4. Got response in **8 minutes 15 seconds**.

#### Pi Lite context

1. Renamed and moved the `~/.pi/agent/prompts/lite.md` file from above to `~/.pi/agent/AGENTS.md`;
2. Pi detected it as a "context";
3. Prompts response time was still the same **8 minutes 15 seconds**.

#### Pi APPEND_SYSTEM

1. Renamed and moved the `~/.pi/agent/AGENTS.md` file above to `~/.pi/agent/APPEND_SYSTEM.md`;
2. Pi did not show anything, but I saw it in Ollama logs;
3. Prompts response time was still the same **8 minutes 15 seconds**.

#### Smaller Ollama context

Previously I set Ollama context size to 16,384 (2^14). I used an environment variable `OLLAMA_CONTEXT_LENGTH=16384`
for that. I assumed that smaller context would improve performance.
So I reduced it by 4096 (2^12) to 12,288. But it did not result in any time improvements.

#### Raspberry Pi 4

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

#### Simplified prompt

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

This response took just 2 iterations according to Ollama logs and I received the result in: 1m 50s + 2m 27s = **4
minutes 17 seconds**
I finally beat the 5 minutes barrier!

#### Context mode

I decided to try [context-mode](https://pi.dev/packages/context-mode) pi extension,
since I assumed smaller context will improve performance even further:

```
pi install npm:context-mode
```

I tested the prompt and got an error in Ollama logs. Well, I should have read its description more carefully.
It did not say that it works with Gemma LLM.

#### Latest Ollama version

I updated to the latest Ollama version 0.30.7 and observed performance improvement by almost a minute
from 4 minutes 17 seconds to **3 minutes 21 seconds**.

#### RTK AI

Next I installed the [rtk-ai](https://github.com/rtk-ai/rtk) extension.
This extension is used when the shell output can be compressed. In my case it had no impact.
It helps in other cases and I often use it at work, so I am still keeping it.

#### Plan Build Git Help

I like the OpenCode approach of switching between modes that tell the LLM if it can or cannot modify the files.
The Aider experience was also still fresh in my head.
I wanted to limit the LLM from editing files and creating pull requests unless I explicitly ask it to do that.
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

# Thank you for reading!
