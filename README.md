# ai-directives

A directory of AI skills, plus core coding directives that apply to every repo.

## Core

`core.md` holds the always-on directives: requirements → tests, TDD, small testable functions, and orchestrator layers.

## Skills

| Skill | Path |
|---|---|
| `dbt` | `skills/dbt` |
| `python-performance-optimisation` | `skills/python/python-performance-optimisation` |
| `python-project-structure` | `skills/python/python-project-structure` |
| `python-testing` | `skills/python/python-testing` |
| `react-best-practices` | `skills/react/react-best-practices` |
| `react-composition-patterns` | `skills/react/react-composition-patterns` |
| `react-native-skills` | `skills/react/react-native-skills` |
| `react-view-transitions` | `skills/react/react-view-transitions` |
| `rust-best-practices` | `skills/rust/rust-best-practices` |
| `rust-testing` | `skills/rust/rust-testing` |
| `shadcn` | `skills/shadcn` |

### Conventions
- Each skill is a folder with a `SKILL.md`. The `name` in its frontmatter matches the folder name and is unique across the repo.
- Supporting docs go in the skill's own `references/` folder and are linked from `SKILL.md`.
- Skills must not contradict `core.md`.
