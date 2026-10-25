# Multi-Agent Waterfall Planner

A multi-agent workflow that simulates a Waterfall software development lifecycle, run on three agent frameworks and three LLMs, with a side-by-side comparison of the results.

## Overview

This project explores whether a team of AI agents can plan a software project. I built a group of LLM agents in which each agent plays one role on a software team, and together they produce a Waterfall project plan from a single customer request. No booking system was built.

I ran the same task with three setups, AutoGen, LangChain and Ollama, each with a different framework, model and way for the agents to interact. Then I compared the runs on execution time, token usage and output quality.

### Case study

All three experiments use the same customer brief:

> **Cloud-Based Intelligent Resource Allocation System for Campus Facilities** (Education domain)
>
> 1. Students and faculty book rooms, labs and equipment online
> 2. Administrators view real-time occupancy dashboards
> 3. AI/ML predicts demand and auto-allocates resources
> 4. Integrates with Google Calendar, Microsoft Outlook and SSO
> 5. Sends notifications for booking confirmations and conflicts
> 6. Generates utilization reports for facility management
> 7. Supports 5,000+ concurrent users and maintains 99.9% uptime

## Experiments

| | Experiment 1 | Experiment 2 | Experiment 3 |
|---|---|---|---|
| Notebook | `Experiment1_AutoGen_GPT4o_mini.ipynb` | `Experiment2_LangChain_Sonnet4.5.ipynb` | `Experiment3_Ollama_Llama3.ipynb` |
| Framework | AutoGen | LangChain | Ollama |
| Model | GPT-4o-mini | Claude Sonnet 4.5 | Llama 3 (local) |
| Approach | Predefined Sequence | Round Robin (next speaker selection) | Sequential Pipeline |
| How it works | Project Manager orchestrates the agents in a fixed order through single-turn calls | Agents take turns in rotation, all reading one shared history (last 3,000 characters) | Agents run one after another, each reading the previous outputs (last 3,000 characters) |
| Agents that responded | 6 (excludes the Customer Proxy, which only sends the request) | 7 (includes the Customer Proxy) | 7 (includes the Customer Proxy) |
| Cost | Moderate (API) | Highest (API) | Free (runs locally) |

## Agent Architecture

```text
Customer Proxy → Project Manager → Requirements Engineer → System Engineer
                                 → Software Developer → Test Engineer → Documentation Engineer
```

All three experiments use the same seven agents:

| Agent | Role |
|---|---|
| Customer Proxy | Sends the problem statement to the Project Manager |
| Project Manager | Receives the request and coordinates each engineering agent |
| Requirements Engineer | Writes use cases, functional requirements and non-functional requirements |
| System Engineer | Writes the system architecture and design document |
| Software Developer | Defines development tasks and SLOC (lines of code) estimates |
| Test Engineer | Writes test cases and testing strategies |
| Documentation Engineer | Writes the end-user and technical documentation plan |

The agents work through the five Waterfall phases: Requirements, Design, Implementation, Testing and Documentation.

### Effort model

Every agent uses the same formulas:

```text
Base Effort      = Work / Productivity Rate
Review Effort    = Base Effort x 10%
Total Effort     = Base Effort + Review Effort
Earliest Start   = Previous Phase Earliest Finish + 1
Earliest Finish  = Earliest Start + Total Effort - 1 (rounded up to a whole number)
```

Each agent ends its response with a fixed-format metrics block, so the results can be compared across the experiments.

## Results

| Metric | Exp 1 AutoGen | Exp 2 LangChain | Exp 3 Ollama |
|---|---|---|---|
| Time (seconds) | 222.9 | 1,476.3 | 566.8 |
| Tokens (approx.) | 13,504 | 37,423 | 10,801 |
| Total text written (characters) | 37,340 | 250,209 | 16,632 |

### Experiment Outputs

Each experiment produced the full set of Waterfall planning artifacts for the project configuration: use cases, functional and non-functional requirements, a system design, development tasks with SLOC and effort estimates, test cases and testing strategies, and a documentation plan. The configuration set the domain, the scope limits (such as the SLOC range), the minimum counts and the effort formulas that the agents had to follow in their roles. How well the agents stayed within that configuration shows the behavior of the model, the interaction approach and the framework behind them. The exact counts are in each notebook's saved output.

Token usage and response length were also analyzed for each agent (approximate). They were compared across three frameworks, three models and three interaction patterns, and the per-agent figures are in each notebook's experiment summary.

## Comparison of Experiments

**Speed and cost**

- **Speed:** AutoGen was the fastest, Ollama was in the middle, and LangChain with Claude Sonnet 4.5 was the slowest.
- **Cost:** Ollama is free because it runs locally. AutoGen with GPT-4o-mini had a moderate API cost, and LangChain used the most tokens, so its cost was comparatively higher.
- On speed and cost together, AutoGen was the best of the API-based options and Ollama was the cheapest.
- **Trade-off:** LangChain with Claude Sonnet 4.5 produced the most elaborate plan, at the cost of the longest run time and the highest token usage.

**Output quality**

The quality of the plans was assessed on four points: correctness, consistency, coherence and completeness.

- **Correctness:** Whether the calculations and estimates in the plan are right, such as SLOC totals, effort and finish days.
- **Consistency:** Whether the values agree across the plan, such as each phase starting the day after the previous one finishes and totals matching their parts.
- **Coherence:** Whether the agents' outputs are logically connected across the SDLC phases.
- **Completeness:** Whether each agent's response is complete, with nothing missing or cut off, and covers what its role was asked to provide.

## Repository Structure

```text
.
├── Experiment1_AutoGen_GPT4o_mini.ipynb    # AutoGen · GPT-4o-mini · Predefined Sequence
├── Experiment2_LangChain_Sonnet4.5.ipynb   # LangChain · Claude Sonnet 4.5 · Round Robin
├── Experiment3_Ollama_Llama3.ipynb         # Ollama · Llama 3 · Sequential Pipeline
├── .gitignore
└── README.md
```

Each notebook has its code and its saved results. The output files generated when the notebooks run are not included.

## Setup

### Prerequisites

- Python 3
- Jupyter Notebook, JupyterLab or VS Code
- An OpenAI API key (Experiment 1), an Anthropic API key (Experiment 2), and Ollama installed locally (Experiment 3)

### 1. Add API keys

Create a `.env` file in the project folder:

```env
OPENAI_API_KEY=your_key_here
ANTHROPIC_API_KEY=your_key_here
```

Do not commit your `.env` file. It is listed in `.gitignore`.

### 2. Set up Ollama (Experiment 3 only)

```bash
ollama pull llama3
ollama serve
```

### 3. Run the Experiments

Open a notebook and run all cells. The first cell of each notebook installs its own libraries:

| Notebook | Packages installed |
|---|---|
| Experiment 1 | `pyautogen<0.4`, `openai`, `tiktoken` |
| Experiment 2 | `langchain`, `langchain-anthropic`, `langchain-core`, `anthropic` |
| Experiment 3 | `ollama`, `requests` |

All experiments use `temperature = 0.3`.

### Troubleshooting

| Problem | Fix |
|---|---|
| `ModuleNotFoundError` | Re-run the first cell of the notebook |
| `401 Unauthorized` or missing key error | Check the key in your `.env` file |
| Ollama connection error | Make sure `ollama serve` is running and the model is pulled |

## Tech Stack

- **Languages and tools:** Python, Jupyter
- **Agent frameworks:** AutoGen, LangChain, Ollama
- **LLMs:** OpenAI GPT-4o-mini, Anthropic Claude Sonnet 4.5, Meta Llama 3
- **Focus areas:** Multi-agent systems, agent orchestration, prompt engineering, context injection, Waterfall SDLC planning, effort estimation

## Contributions

I built Experiment 1 (AutoGen) and my two teammates built Experiments 2 and 3. The comparison covers all three.

## Future Work (Phase 2)

Phase 2 will be carried out using the Scrum methodology for iterative development.

- Transition from Waterfall to Scrum methodology for iterative development.
- Define Scrum roles and convert Phase 1 outputs into a structured sprint backlog.
- Plan short sprint cycles and refine agent prompts for improved efficiency.
- Compare Agile and Waterfall performance in terms of quality and execution time.