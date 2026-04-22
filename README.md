```
██████╗ ███████╗ █████╗ ██████╗ ██╗   ██╗      ∩___∩
██╔══██╗██╔════╝██╔══██╗██╔══██╗╚██╗ ██╔╝     |     |
██████╔╝█████╗  ███████║██████╔╝ ╚████╔╝      | •_• |
██╔══██╗██╔══╝  ██╔══██║██╔══██╗  ╚██╔╝       |  ω  |
██████╔╝███████╗██║  ██║██║  ██║   ██║       /|     |\
╚═════╝ ╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝      (_|     |_)
```
Welcome to BEARY, the agentic workflow for Background Research!

## Quickstart

```bash
npx skills add sally-jankovic/BEARY
```

Add this to your project `.gitignore`:

```gitignore
beary-scratchpad/
```

Summon BEARY:

```text
BEARY
```


# What Beary Does
Beary is an agentic workflow that takes a simple prompt and performs internet research, takes notes, and compiles its findings into a whitepaper, complete with citations. You can customize both the audience Beary writes for as well as the sources he looks at.

Currently, Beary as designed is only supported in Cascade/Windsurf, but you are super welcome to tweak it to fit your preferred agent!

# Before you use
Remember that Beary is agent-powered, and LLMs hallucinate. For important topics, please check references and citations to ensure that they are
1) Real
2) Accurate
3) Up-to-date

It is highly encouraged to choose models that have a better track record of being accurate and up-to-date.  It is *never* appropriate to steal ideas or commit plagiarism or to pass this work off as your own research. Please cite use of AI whenever applicable.


# How to Use Beary
1. Install using Quickstart above.

2. Summon BEARY by calling BEARY in your prompt.

```text
BEARY, research <topic> and create a whitepaper with citations
BEARY, deep-dive <topic>
BEARY, prepare a cited research brief on <topic>
```

3. Provide topic details when prompted:

```text
TOPIC: my-topic
Description: What this topic is about.
Purpose (Optional): What you want to achieve with this topic or why you are interested in it.
```

4. Confirm workflow options:

```text
Mode: Hibernation or Hyperphagia
Review flow: Attended or Unattended
```

5. BEARY will run research in `beary-scratchpad/{TOPIC}` and publish final deliverables to your configured output path (default: `whitepaper-output/`).


# Getting Started

## USER.md setup (interview-first)
On first run, BEARY will interview you and generate/update:
- `.agents/skills/beary/USER.md`

It captures defaults for:
- Audience level and role
- Purpose style (exploration vs decision support)
- Source priorities and exclusions
- Freshness preference (e.g., 30/60/90 days)
- Depth and contradiction strictness
- Output path

### Configuring Output Path manually (optional)
You can still edit `.agents/skills/beary/USER.md` directly:

```markdown
<!-- OUTPUT_PATH: whitepaper-output -->
<!-- OUTPUT_PATH: completed-research -->
<!-- OUTPUT_PATH: /Users/username/Documents/beary-research -->
```

Path rules:
- Relative paths are resolved from project root
- Absolute paths are required outside the repo
- Invalid paths fall back to `whitepaper-output/`

Recommended: add this to your project `.gitignore` to keep temporary research workspace out of commits:

```gitignore
beary-scratchpad/
```

## Sample prompt
```text
BEARY, research the topic below and produce a cited whitepaper.
TOPIC: Multi agent setups
Description: What are the current setups, tools, and services for multi-agent systems? How do their abilities compare as personal AI assistants and coding setups vs in production? How are they being benchmarked? What are the cost trademarks?
Purpose: I am trying to learn more about multi-agent setups, where the industry is, and where and how I can begin experimenting with them and evaluating them.
```
## Command Approval
During the workflow, Cascade may prompt you to approve terminal commands (e.g., creating directories). Most of these are safe operations that have been marked with `// turbo` annotations to auto-run.

If you still see approval prompts for safe commands like `mkdir`, you can configure your IDE to auto-approve them:
1. Open Windsurf Settings
2. Navigate to the command allowlist settings
3. Add safe commands (like `mkdir`) to your allowlist

This is a **per-IDE setting**, so each user needs to configure it individually if they want to skip approval prompts beyond what the workflow annotations cover.

## Model Selection
Windsurf AI uses *one request* per action. This means that it will not use multiple requests to generate a response. Since there is one prompt and a few responses, *it will only take a few credits to run the research workflow.* Using high-cost models is therefore encouraged, as it will produce a higher-quality whitepaper for relatively little.