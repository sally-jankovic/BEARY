```
██████╗ ███████╗ █████╗ ██████╗ ██╗   ██╗      ∩___∩
██╔══██╗██╔════╝██╔══██╗██╔══██╗╚██╗ ██╔╝     |     |
██████╔╝█████╗  ███████║██████╔╝ ╚████╔╝      | •_• |
██╔══██╗██╔══╝  ██╔══██║██╔══██╗  ╚██╔╝       |  ω  |
██████╔╝███████╗██║  ██║██║  ██║   ██║       /|     |\
╚═════╝ ╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝      (_|     |_)
```
Welcome to BEARY, the agentic workflow for Background Research!


# What Beary Does
Beary is an agentic workflow that takes a simple prompt and performs internet research, takes notes, and compiles its findings into a whitepaper, complete with citations. You can customize both the audience Beary writes for as well as the sources he looks at.

Currently, Beary as designed is only supported in Cascade/Windsurf, but you are super welcome to tweak it to fit your preferred agent!

# Before you use
Remember that Beary is agent-powered, and LLMs hallucinate. For important topics, please check references and citations to ensure that they are
1) Real
2) Accurate
3) Up-to-date

It is highly encouraged to choose models that have a better track record of being accurate and up-to-date.  It is *never* appropriate to steal ideas or commit plagiarism or to pass this work off as your own research. Please cite use of AI whenever applicable.


# Install Beary (skills.sh)

Install BEARY into any project as a single skill package:

```bash
# from BEARY repo root
scripts/skills.sh install /path/to/your/project
```

Update an existing installation:

```bash
scripts/skills.sh update /path/to/your/project
```

Check installation health:

```bash
scripts/skills.sh doctor /path/to/your/project
```

Uninstall:

```bash
scripts/skills.sh uninstall /path/to/your/project
```

# How to Use Beary
1. Install via `scripts/skills.sh install /path/to/your/project` (or clone/copy manually).

2a. In your cascade terminal, run the slash command:
`/research-to-whitepaper`.
2b. Then provide your topic details:
TOPIC: my-topic
Description: What this topic is about.
Purpose (Optional): What you want to achieve with this topic or why you are interested in it.
2c. Press enter

3. The workflow will prompt you before beginning:
a. If you do not provide a purpose, the agent will prompt you for one. You may skip it but it is recommended to give your whitepaper a narrative throughline.
b. Choose between Hibernation mode (token-conservative) vs Hyperphagia mode (token-generous)
c. Choose between Attended (you approve research before it begins writing) vs Unattended (it runs with no further user contact)


# Getting Started

## Set USER.md preferences
In order to direct Beary, you need to set your preferences in the `.agent/USER.md` file for:
- **Intended Audience** (e.g., engineer, data scientist, racing hobbyist, etc.)
- **Desired Sources** (e.g., code repositories, academic papers, blogs, etc.)
- **Output Path** (optional): Where completed whitepapers should be moved after the workflow completes

### Configuring Output Path
By default, completed whitepapers are moved to `whitepaper-output/` (which is gitignored). To customize this:

1. Open `.agent/USER.md`
2. Find the "Output Path" section
3. Update the path in the comment: `<!-- OUTPUT_PATH: your-custom-path -->`

**Path Types:**
- **Relative paths** (e.g., `completed-research`, `docs/whitepapers`) are relative to the project root
- **Absolute paths** (e.g., `/Users/username/Documents/research`) **MUST** be used for directories outside the repository
- If the path fails to resolve, the workflow will fall back to `whitepaper-output/` and notify you

Examples:
```markdown
<!-- OUTPUT_PATH: completed-research -->
<!-- OUTPUT_PATH: /Users/jankovis/Documents/beary-research -->
```

This keeps your workspace clean and prevents accidentally committing research artifacts.

## Sample prompt
```/research-to-whitepaper TOPIC: Multi agent setups
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
Windsurf AI uses *one request* per action. This means that it will not use multiple requests to generate a response. Since there is one prompt and a few responses, *it will only take a few credits to run the research-to-whitepaper workflow.* Using high-cost models is therefore encouraged, as it will produce a higher-quality whitepaper for relatively little.