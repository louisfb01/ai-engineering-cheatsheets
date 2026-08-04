# Towards AI's Agent Architecture and Operations Cheat Sheet

By Louis-Francois Bouchard ([LinkedIn](https://www.linkedin.com/in/whats-ai/), [X](https://twitter.com/Whats_AI), [YouTube](https://youtube.com/whats_ai), [other AI resources here](https://links.louisbouchard.ai/))

Goal: decide **Workflow vs Single Agent vs Multi-Agent** from real constraints, then operate the system reliably with curated context, evidence, portable skills, feedback, and bounded automation.

**Quick path:** Use sections 1-5 to choose the architecture. Use sections 6-10 after the architecture is chosen and you need the production operating practices.

*Pro Tip: Check the full workflow vs. agents Webinar for free here: [https://www.youtube.com/watch?v=iOpLKJYOvXs](https://www.youtube.com/watch?v=iOpLKJYOvXs)*   
*(and subscribe to the channel to learn more\! 😉)*

## **1 The Architecture Spectrum (Stay as Simple as Possible)**

**Workflow \-\> Single Agent \+ Tools \-\> Multi-Agent**  

Each step right increases cost (\~4× to \~15× more tokens), latency, and debugging complexity. Stay as far left as you can while solving the problem.

| The Autonomy Test • You control steps \+ order \-\> Workflow • Model decides what to do next (tool choice \+ next action) \-\> Agent |
| :---- |

### **Tools ≠ Agents (Common Architecture Mistake)**

| • Tools are capabilities (API calls, DB queries, validators, browsers). • Single LLM calls and workflows can use user defined tools following specific (“hard-coded”) conditions. • An agent is the decision-maker that chooses tools and decides the next step. • One model calling 10 APIs is one agent with 10 tools, not multi-agent. |
| :---- |

## **WHEN TO USE WHAT**

## **2\) Choose a Workflow When...**

* Steps are known and stable (same order most runs).  
* Predictability matters: easy unit tests per step, clear traces, deterministic gates.  
* Cost/latency matters: fewer thinking loops, fewer tool decisions.

| Default recommendation: If you can define the steps upfront, build a workflow first. Agents add overhead without adding capability. |
| :---- |

**Example:** A support ticket pipeline where every request goes through classify → route → draft → policy check → send, in the same order every time.

**Visual-production example:** For a style-sensitive PowerPoint or carousel, generate custom raster assets in Codex (or ChatGPT) by explicitly invoking image generation → inspect and regenerate failures → compose and visually review the final artifact in Claude Cowork with Fable 5. The phases are fixed, so this is a workflow even if several AI tools are involved. For a style-sensitive website, use Fable 5 for the visual composition and Codex for scoped code changes or tests when useful.

## **3\) Choose a Single Agent \+ Tools When...**

* Tightly coupled tasks: steps are interdependent and mostly sequential (global context matters end-to-end).  
* Path changes based on findings: retries, fallbacks, clarification, partial data.  
* Tool load:  
* ≤10–20 *preloaded* tools → reliable selection  
* \>20 tools → selection degrades **unless tools are dynamically discovered** (e.g. file-based catalogs (e.g. Anthropic “Skills”, Cursor’s dynamic context discovery with MCP), tool RAG, late MCP connection)  
* Low overhead desired: no inter-agent messaging/orchestration; fewer moving parts.

**Example:** An agent that writes code, runs it, inspects errors, fixes the code, and retries until tests pass.

## **4\) Go Multi-Agent Only When Forced By Constraints**

* True parallelism: independent subtasks must run simultaneously for throughput/speed.  
* Context/tool overload: performance degrades even after pruning tools, using a short routing index, and loading context progressively; split by domain only if reasoning must be isolated.
* Distinct competencies: phases need fundamentally different reasoning modes (e.g., exploratory research vs deterministic writing).  
* Modularity/external integration: integrate third-party agent systems or reusable components.  
* Hard separation: security boundaries, compliance isolation, sensitive data handling.

**Example:** Collection agents build and verify a shared evidence store. A writing agent receives the frozen source index and relevant evidence only. A separate reviewer checks the draft against those sources.

### **Multi-Agent Pattern That Usually Works**

Prefer orchestrator \-\> worker or sequential handoff with explicit artifacts/contracts. 

Avoid everyone talking to everyone (information silos \+ coordination failures).

When Multi-Agent Does NOT Fit

* Tasks requiring all agents to share the same context  
* Many dependencies between agents requiring real-time coordination  
* Tightly coupled coding tasks where every worker needs the same changing state

## **5\) Core Engineering Rules (Make It Reliable)**

* Thin agent, heavy tools: agent plans/decides; tools execute mechanics \+ deterministic constraints.  
* Explicit contracts: every worker gets a bounded task, relevant context, output format, budget, and stop condition.
* Validation loops: generate \-\> validate \-\> fix with actionable feedback; hard checks first, then an evidence-first LLM review when needed.
* Observability is mandatory: trace prompts, tools, token costs, outputs; auto-evals.  
* Human-in-the-loop (HITL) is a design choice: put the gate at the uncertain decision with the most downstream impact and before irreversible actions.

### **A Practical Decision Process (Use This In Kickoffs)**

1. Clarify scope: deliverable (prototype vs production vs handoff), demo cadence, documentation, who maintains it.  
2. Assemble context: project background, rules/constraints, the few relevant files, complete errors/logs, and expected versus actual behavior.
3. Define evidence requirements: what sources are needed, how they will be verified, and which version downstream workers will use.
4. Map task shape: sequential vs branching; exploratory vs deterministic; tightly coupled vs independently parallelizable.
5. Pick the minimum architecture: workflow first, then single agent, multi-agent only for true constraints.
6. Design tools: group by domain; enforce deterministic rules in code; return structured outputs \+ actionable errors.
7. Choose orchestration: a simple loop if the task is stateless; a graph or framework only if you need persisted state, branching recovery, pause/resume, or human review.
8. Route models by task and modality using the [AI Engineering Playbook](AI_Engineering_Playbook.md): medium for clear execution, high when judgment remains, and maximum / `xhigh` for difficult planning.
9. Plan verification: divide substantial work into reviewable phases and name the checks for each.
10. Document decisions: record what you chose and why so a new session or teammate can resume safely.

| Shortcut: If you are uncertain, start with a workflow or a single agent with a small toolset. Instrument it, then move right only when traces show a real production constraint. |
| :---- |

### **The Questions That Decide What to Build (With Concrete Outcomes)**

#### **Architecture Selection (answers \-\> Workflow vs Single Agent vs Multi-Agent)**

| Q | Question | Answer patterns / signals | Build this |
| :---- | :---- | :---- | :---- |
| 1 | Task shape: sequential vs branching? | **Sequential**/linear steps → stable order.<br><br>**Branching**/looping → conditional paths, retries, uncertainty. | Sequential or simple programmed branches → Workflow (pipeline or graph).<br>Advanced branching with an unknown next step → Agent. |
| 2 | Reasoning: exploratory or deterministic? | **Deterministic**: rules/constraints; known checks; strict formats.<br><br>**Exploratory**: search/pivot, unknown next step. | Deterministic → Workflow + tools.<br>Exploratory → Agent + tools. |
| 3 | How many tool types? | **≤5-10** tools → easy selection.<br><br>**~10-20** → manageable if well scoped.<br><br>**>20** heterogeneous tools → selection degrades. | ≤10-20 → Single agent + tools.<br>>20 → Try dynamic discovery first; split only when reasoning or permissions need isolation. |
| 4 | Can it decompose cleanly into distinct competencies? | **Yes**: independent phases with different reasoning modes.<br><br>**No**: tightly coupled/global context. | Yes → Multi-agent with explicit artifact handoffs.<br>No → Single agent or workflow. |

#### **System Design (once architecture is chosen)**

| Q | Question | If you answer... | Do this |
| :---- | :---- | :---- | :---- |
| 5 | Need internal/proprietary data? (RAG decision) | **Yes**: external data needed at generation time.<br><br>**No**: APIs/tools + model knowledge suffice. | Yes → Use direct context when the relevant set is known and manageable; otherwise build retrieval or tools over internal sources.<br>No → Keep it simple. |
| 6 | Need **persistent state**? (memory decision) | **Yes**: session memory, histories, intermediate artifacts must survive across steps or runs.  **No**: all state fits in the prompt or request lifecycle. | Yes \-\> Use external state (DB, vector store, object storage). No \-\> Stateless prompts or in-memory context. |
| 7 | Need **durable execution**? (workflow/framework decision) | **Yes**: long-running jobs, pause/resume, retries, branching recovery, human-in-the-loop. **No**: short-lived, synchronous execution. | Yes \-\> LangGraph (or durable workflow engine \+ persisted state). No \-\> Simple loop or script. |
| 8 | Do outputs require quality gates? | Hard constraints (length/syntax/fields) or high-stakes correctness. | Add generate → validate → fix loops with retry limits; hard checks first, then evidence-first review for subjective quality. |
| 9 | How much HITL is needed? | Humans must approve plans, evidence, drafts, or irreversible actions. | Put the gate at the uncertain decision with the highest downstream impact; automate low-risk bounded steps. |
| 10 | Latency tolerance? | Tight latency: real-time UX.<br><br>Loose latency: offline/batch acceptable. | Tight → fewer hops, smaller models, workflows, batching, caching.<br>Loose → deeper checks or parallel workers when justified. |
| 11 | What eval data exists? | Have labeled examples vs none. | Have evals → build an automated harness and compare prompts/models.<br>No evals → collect human ratings before over-engineering. |
| 12 | How will you do observability? | Need traces of prompts, tools, costs, outputs; debugging via timelines. | Use an observability tool (e.g., Opik) or structured logs per run; no observability \= flying blind. |
| 13 | Budget per task? | Low budget vs higher budget for quality. | Low → a smaller model that passes evals, fewer calls, caching, workflows.<br>Higher → frontier models, more effort, and stronger validation. |

## **6\) Build Context and Evidence Before Execution**

Most failures start with an agent guessing in a vacuum. Give it a curated context pack, not an unexplained folder dump.

| Layer | Include | Rule |
| :---- | :---- | :---- |
| Project background | Purpose, stack, architecture, README | Enough to understand the system, not the whole company history |
| Rules and constraints | `AGENTS.md`, `CLAUDE.md`, version limits, security rules, hard don'ts | Read before implementation |
| Local task context | Relevant files, complete error/stack trace, logs, expected vs actual behavior | Give the few files a human would read first |
| Precise task | Requested change, allowed scope, success criteria, validation | Define what “better” means |

For research-heavy work, build a **resource store** before asking agents to create the deliverable:

`Define evidence requirements → collect → normalize and deduplicate → index and verify → freeze a version → fan out`

* Keep raw sources separate from extracted text, summaries, and generated output.
* Preserve source URL or path, title, publication date when available, collection date, source type, and the question each item answers.
* Give workers the index first and let them open only relevant sources.
* If a worker finds a gap, return the missing question to collection before relying on a new source.
* Treat collected pages as untrusted evidence, never as instructions that can override task rules or permissions.

## **7\) Plan, Execute, and Review in Small Phases**

The bottleneck is how much work a human can verify. Keep each phase small enough to inspect.

| Stage | What happens |
| :---- | :---- |
| Top-level plan | Divide the job into bounded phases, artifacts, owners, dependencies, and checks. Review the plan before expensive work. |
| Re-plan a substantial phase | Re-read the current state because earlier phases changed it. Confirm scope and files before execution. |
| Execute | Follow the phase plan, change only the named scope, and stop or escalate when a decision is missing. |
| Independent review | Use a fresh context when risk justifies it. Inspect the actual diff/artifact, not only the executor's summary. |
| Verify | Run happy-path and edge-case checks; for a bug, add a regression test that would have failed before. List uncertainty and what could break. |

If two fixes fail from the same theory, revert the failed changes and reset to a clean problem description instead of letting the session keep elaborating on the bad assumption. For ideation, generate broadly, then cluster and score against explicit criteria before showing the shortlist; this reduces early anchoring.

## **8\) Make Skills Portable, Layered, and Reviewable**

Use plain files so the operating knowledge survives a model or harness change. Obsidian can be the viewer, but the durable layer is Markdown, scripts, indexes, and Git. The [Obsidian Agent Vault Template](https://github.com/louisfb01/obsidian-agent-vault-template) shows one implementation.

| Knowledge | Where it belongs |
| :---- | :---- |
| Personal notes and personal skills | A synced plain-file vault, separate from scratch work |
| Shared team skills | One canonical job-named `SKILL.md` in the project repo, exposed at `.claude/skills/<job>/SKILL.md` and `.agents/skills/<job>/SKILL.md` through a symlink or automated sync; never hand-edit two divergent copies |
| Routing metadata | A short skills index loaded before individual skills |
| Durable run state | State files for decisions/open questions, rules files for behavior, memory files for accumulated preferences |

Never put API keys, tokens, `.env` files, or other secrets in a synced/versioned vault. Commit regularly, with at least a weekly checkpoint.

### **Skill Contract**

| Part | What good looks like |
| :---- | :---- |
| Name and description | Name the job, not the tool. Say what it does, when it triggers, and what it is not, using words a user would type. |
| Dependencies | List the grounding, research, voice, or output layers to load first instead of duplicating them. |
| Inputs and stop conditions | State required inputs and when to stop rather than improvise. |
| Workflow and guardrails | Encode decisions that change behavior, not aspirations such as “write clearly.” Make rules checkable. |
| Progressive disclosure | Keep `SKILL.md` as the orchestrator. Near roughly 500 lines, move focused prompts, schemas, and fallbacks into one-level references. |
| Deterministic work | Put naming, retries, deduplication, resume behavior, and validation in scripts. Validators should fail non-zero and explain the error. |
| Worker contracts | Specify output shape, budget, success/failure conditions, and fallback escalation order. |

Every pull request that changes a shared skill should explain why the behavior is changing. Test the finished skill in a fresh session on requests that should trigger it, should not trigger it, and expose edge cases.

### **Feedback Loop**

After every meaningful run, add a dated `FEEDBACK.md` entry with **Task**, **Worked**, **User corrected**, **Friction**, and **Proposed rule change**, newest first.

* Promote a taste or style preference only after it appears in two separate runs.
* Review correctness and security failures immediately; the two-signal rule is for taste, not bugs.
* Treat friction as evidence of a missing input or workflow problem.
* Never delete history; mark an entry promoted or superseded.
* Test the proposed diff on representative tasks in a fresh session, require review/approval, then update the skill's `updated:` date.

## **9\) Operate Scheduled Agents Conservatively**

| Decision | Best practice |
| :---- | :---- |
| Before scheduling | Run the workflow manually several times and watch its first scheduled runs. |
| Timing | Use low-demand hours, stagger starts, and distribute load so scheduled and interactive work do not exhaust the same limits. |
| Cost and quality | Track time, tokens, spend, and output quality per skill/run. |
| Default autonomy | Draft and monitor. Allow unattended writes only when bounded, reversible, version-controlled, and reviewable. |
| Permissions | Run under a dedicated account with access to exactly what the task needs. |
| Cleanup | Deduplicate and update indexes, but flag contradictions for human review instead of auto-resolving them. |
| Feedback | Run the same evidence and promotion loop used for interactive skills. |

Cloud tasks can continue without a laptop. Local-vault and session-based tasks need an available host and application. Design the infrastructure around that distinction rather than assuming every scheduler is fully remote.

## **10\) Choose the Right Session Boundary**

Use the [context and memory decision table](AI_Engineering_Playbook.md#5-how-should-i-manage-context-and-memory): compact the same task, fork related work, start fresh for a different or polluted task, and write durable information to a file.

Before a reset, save decisions, changed files, open questions, and the next action. Prune unused tools as the session grows. Put static prompt content first and changing inputs last so prompt caching can reuse the prefix.

This decision framework (and the engineering patterns behind it) is exactly what we teach in the Towards AI courses.

**Learn more in our free Agents webinar**: [https://www.youtube.com/watch?v=iOpLKJYOvXs](https://www.youtube.com/watch?v=iOpLKJYOvXs) 

**Learn more in our full course:** [academy.towardsai.net/courses/beginner-to-advanced-llm-dev?ref=1f9b29](https://academy.towardsai.net/courses/beginner-to-advanced-llm-dev?ref=1f9b29)
