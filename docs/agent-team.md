# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate Mona's Project Pulse dashboard team.

| Agent | Target model | Responsibility | Agent definition |
| --- | --- | --- | --- |
| **Orchestrator** | Claude Opus 5.5 (copilot) | Coordinates the Planner, Coder, and Designer; turns the implementation plan into sequential or parallel phases, assigns file scopes, and verifies the integrated dashboard. | [.github/agents/orchestrator.agent.md](../.github/agents/orchestrator.agent.md) |
| **Planner** | Claude Opus 5.5 (copilot) | Researches the codebase and requirements, identifies dependencies and edge cases, and produces an ordered implementation plan with file assignments and validation expectations. | [.github/agents/planner.agent.md](../.github/agents/planner.agent.md) |
| **Coder** | GPT-5.5 (copilot) | Implements the dashboard structure and behavior, creates support configuration when assigned, and validates deterministic, testable changes. | [.github/agents/coder.agent.md](../.github/agents/coder.agent.md) |
| **Designer** | Gemini 3.7 Flash (copilot) | Designs the dashboard UI/UX, including accessibility, responsive layout, visual hierarchy, project cards, status badges, priority treatment, and polished interaction affordances. | [.github/agents/designer.agent.md](../.github/agents/designer.agent.md) |

## How the team will work together

The Orchestrator starts by asking the Planner to research Project Pulse and produce an implementation plan with ordered phases, file ownership, dependencies, edge cases, and validation expectations. The Orchestrator then assigns the work to the Designer and Coder in scopes that do not overlap without a dependency.

The Designer creates the dashboard's visual direction and responsive, accessible UI in its assigned design files. The Coder implements the dashboard structure and behavior in its assigned code files and creates required support configuration. They work in parallel when their file scopes are independent, or in sequence when one phase depends on the other's output.

GitHub Copilot CLI in this Codespace will orchestrate delegation and integration, verify that the combined result hangs together, and report progress and blockers. The agents do not stage, commit, or push changes; those Git operations remain with the learner.
