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
Beary is an agentic workflow that takes a simple prompt and performs internet research, takes notes, and compiles its findings into a whitepaper, complete with citations.

Currently, Beary as designed is only supported in Cascade/Windsurf, but you are super welcome to tweak it to fit your preferred agent!

# Before you use
Remember that Beary is agent-powered, and LLMs hallucinate. For important topics, please check references and citations to ensure that they are
1) Real
2) Accurate
3) Up-to-date

It is *never* appropriate to steal ideas or commit plagarism. Please cite use of AI whenever applicable

## WWT Usage
This agent workflow was developed by a WWT Management Consulting employee for other Management Consulting employees. While you are free to use it for other use, when using it for work, *YOU MUST CONSULT THE WWT AI MC HANDBOOK BEFORE USAGE*.

### Appropriate Work Use Cases
This list is not exhaustive, but is merely to give you an idea of what is appropriate.
1. You have to attend a talk/conference/meeting about something you don't know much about and you would like a rundown of all the big topics in the field.
2. You are on a busy project and want to keep an overall eye on industry trends.
3. You need to give a presentation or get assigned to a project and don't know where to start. You use Beary to get a medium-depth overview of the topic, and follow up on sources as well as consult coworkers with expertise.
4. Your boss asks you for some background on a low-stakes topic. You ask her (or him, I guess) if you can use AI to do the heavy lifting and she is fine with it.
5. You have to scope out a complicated solition to a problem and want an overview of the industry trends and best practices. Beary's paper is your starting point.

### Inappropriate Use Cases
This list is not exhaustive, but is merely to give you an idea of what is inappropriate.
1. You are tasked with your first presentation as an analyst. You ask Beary to do all the research and copy it verbatim into your presentation.
2. Someone asks you to write a whitepaper for the WWT website about something you're an expert on. You know you know about the topic, but you're busy. So you ask Beary to do it, and pass it off without mentioning that Beary did the heavy lifting (Beary's feelings *will* get hurt, and he doesn't even have feelings)
3. You are a mid-level engineer with solid experience but working in an unfamiliar tech stack and your dog has been eating your sofa, which is stressing you out. You need to make some quick game-time decisions about what to do from an architecture standpoint and outsource it to Beary. You offer Beary's proposed solution after a single skim of the whitepaper, and don't even consult that data engineer who did the exact same thing for their last project.

# How to Use Beary
1. Clone repository locally or copy files into your own Cascade project.

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


# Tips and Tricks

## Sample prompt
```/research-to-whitepaper TOPIC: runai-gpuaas-airgapping
Description: How to make state-of-the-art workflows for GPUaaS using RunAI for heavily airgapped environments, plus what the industry standards, current best practices, and gaps are.
Purpose: I am trying to take a heavily manual and sub-optimal approach to loading self-hosted models into our self-hosted version of RunAI. I need to establish governance and automation, as well as make a re-usable workflow that will help create a GPUaaS that will allow LLMs to be "on-demand" (or as close as possible to "on-demand") for experimentation and iteration.
```
## Model Selection
Windsurf AI uses *one request* per action. This means that it will not use multiple requests to generate a response. Since there is one prompt and a few responses, *it will only take a few credits to run the research-to-whitepaper workflow.* Using high-cost models is therefore encouraged, as it will produce a higher-quality whitepaper for relatively little.