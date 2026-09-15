# seft-quiz

Self-study quiz site (Multiple Choice + Q&A tabs, see `.claude/skills/rs-gen-quiz/`).

## Content Scope

All quiz topics are scoped to the **Software Engineer major** — Technical or Behavioral only.

- **Technical** — CS fundamentals, languages, frameworks, infrastructure, tools (e.g. algorithms, system design, databases, Kubernetes, Datadog).
- **Behavioral** — SWE interview/career topics (e.g. STAR method, conflict resolution, leadership principles, communication).

When scoping a vague `/rs-gen-quiz` topic, narrow it into one of these two — not an unrelated domain.

## Categories (`index.html`)

`index.html`'s `MANIFEST` groups every generated quiz under a fixed set of categories — the same list `/rs-gen-quiz` files new quizzes into:

- **Backend** — languages, frameworks, APIs, databases
- **Infrastructure** — cloud, Kubernetes, Terraform, observability/monitoring (Datadog, etc.)
- **AI** — ML/LLM fundamentals, agents, RAG, prompting
- **System Design** — architecture, scalability, distributed systems
- **Behavioral** — see Content Scope above

Adding a 6th category means updating both this list and `index.html`'s `MANIFEST` together — don't add one without the other.
