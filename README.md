# llm-answer-ruleset

A system file that constrains how an LLM answers, plus six skills built independently and used under it.

## Scope & Limitations

**Most of the answer was decided before this file was read. In order of importance: Prompt Engineering (the shape of the question/prompt itself), Model selection and effort level, account for the bulk of what comes back.** A standing instruction file operates on the remainder, shifting a distribution rather than overriding what training baked in. Two separate sessions reached that conclusion independently. Treating the file as the primary lever is a category error, it is a trim tab and not a rudder. The realistic gain is marginal and mostly visible in user-specific usage by eliminating repetition and aligning soft behavioral traits across sessions. Combined with other common good practices gives the optimal results.

**Neither party can verify compliance from inside a conversation.** You cannot tell the difference on something you do not know, and neither can the model. Genuine convergence and conciliatory drift produce identical-looking text. The `TIER MARK` and the graded `BASIS` exist to make the reasoning legible enough for the drift to surface so the reader can catch it themselves, which is the actual mechanism, not self-enforcement.

**Generated context files may not help.** A February 2026 ETH Zurich study (AGENTbench, 138 instances across 12 Python repos) found LLM-generated context files lowered task success ~3% versus no context, while raising inference cost >20%, hand-written files improved success ~4%. Single study, one language, small n, directional rather than settled, but it argues against asking a model to scaffold a file like this for you.

**Untested against a corpus.** The tier boundaries, particularly T3/T4, were cut by inference and never back-tested on real misclassifications.

**One known context-cost trap.** IF, held in a preferences field AND pasted as a document, the ruleset is paid for twice per turn.

**Limits no instruction removes.** Properties of the mechanism, not gaps in the file:

- Instructions are context, not enforcement: Anthropic's own documentation says a `CLAUDE.md` is treated as context, with no guarantee the model follows it. Only a hook or a permission rule enforces anything. In practice compliance is better than that suggests, but a tendency is not a guarantee.
- Stated reasoning is not evidence of a correct answer: the explanation of why an answer was produced is generated the same way the answer was. It can read as sound and still miss the actual logic, because it is not read off the computation. `RECONSTRUCT` and `BASIS` are aids for the reader, not audits.
- Self-grading is internal: whatever assigns established, contested, inferred, or speculative is the same system that made the claim. A confident error grades itself confident.
- Hallucination is a property of the mechanism: next-token prediction under uncertainty produces fluent continuations whether or not grounding exists. `UNKNOWN` lowers the rate, nothing at the prompt layer takes it to zero.
- Adherence decays with length, both of the thread and of the file: Anthropic recommends keeping such a file under 200 lines. `RULESET.md` is 166, so every addition now trades against compliance with what is already there.
- Nothing transfers cleanly between models: the rules were cut by watching one model fail. A different model, especially a smaller one, fails differently and follows long instructions less reliably.
- Retrieval needs tools: T1 says retrieve rather than recall. With no search or fetch tool available the model cannot comply, and the tier quietly degrades to recall against a training cutoff.

## Install

Two deployments. Pasted into whichever field your assistant provides for standing instructions:

```
RULESET.md  →  Claude: Settings › Profile › personal preferences
               ChatGPT: Settings › Personalization › Custom instructions
               API: the system parameter
```

Or placed as a `CLAUDE.md`, which Claude Code loads on its own at the start of every session:

```
ln -s /path/to/RULESET.md ../CLAUDE.md
```

Claude Code concatenates `CLAUDE.md` from the working directory and every directory above it, so one sitting a level above your repositories governs all of them. At `~/.claude/CLAUDE.md` it governs every project on the machine. Symlink a project-level file rather than copying it, but keep the user-level one a real file, Claude Code skips a symlinked `~/.claude/CLAUDE.md` in Cowork desktop sessions.

It was written against and used with Claude, but nothing in it is Claude-specific except the skills below.

On `AGENTS.md`: the slot is shared, the job isn't. That convention carries repository operations, build commands, test procedures, files not to touch, scoped to work on one codebase. This carries answer governance, scoped to any question asked. Putting it in the agent-instruction slot works and is how it's used here, it is just an unusual thing to put there. Claude Code reads `CLAUDE.md` and not `AGENTS.md`, so a repository keeping the latter for other tools needs a `CLAUDE.md` containing `@AGENTS.md` to pull it in.

### Skills

`skills/` holds six Claude skills, packaged as `.skill` archives. Each unpacks to a `<name>/` directory holding a `SKILL.md`, plus `references/` and `assets/` where the skill uses them.

```
unzip 'skills/*.skill' -d ~/.claude/skills/
```

Or install one at a time by dropping a `.skill` file into Claude directly.

| Skill | What it does |
|---|---|
| `ainews-digest` | Restructures a news.smol.ai issue into a tiered HTML screening document |
| `exam-cram` | Builds a cram document from course materials — past-paper mining, mark-weighted triage, dated plan |
| `git-commit-message` | Composes Conventional Commits messages from a diff. Composes only; never stages or commits |
| `github-readme` | Writes or audits a README, tuned for personal tools and CLI scripts |
| `youtube-signal-summary` | Extracts a video's content from its transcript at maximum signal per minute read |
| `youtube-video-screener` | Triages videos into watch / read-summary / skip |

## How the two parts relate

The ruleset governs skill execution by being in the system prompt on every turn. Skills therefore do **not** reference it. A skill is loaded on trigger and read in isolation. A skill needing behavior the ruleset forbids has to state the exception in its own file.

## How it works

Four sections, each doing one job.

**TRUTH** classifies the question before answering it, on *how the answer is verified* rather than how hard it is. T1 retrieved from sources, T2 derived or executed, T3 constructed from converging evidence, T4 weighed where evidence doesn't close, T0 generative with no truth value. The classification fixes both the method and how much of the route the answer shows.

**ACCURACY** applies at every tier. Scrutinize the premise before answering it, ask rather than fill a gap from priors, test the answer against its strongest counter, let the verdict track the premise rather than a target rate of agreement or dissent, grade the grounds, state absence of knowledge instead of reconstructing fluently around it.

**DELIVERY** governs form. Tier marker, direct answer first, rules applied silently, conciseness as an invariant rather than a default, and no stylistic device used often enough to become a fingerprint.

**DOMAIN** holds the triggered rules: a two-part spec gate before building anything new, production defaults for generated code, and a comment register.

Plus `!commands` for single-turn or toggled mode changes, `--flags` for modifiers, and an `AMENDMENT` clause admitting a new rule only when it names a behavior no existing rule covers.

## Good prompting practices

The document is one relatively small coefficient, to be used alongside common practice and not as a replacement. Most of the answer comes from other factors, and using those correctly does far more than the document ever could.

- Accuracy versus precision: you can dictate accuracy, precision is mostly bounded by the model itself and by hard physical constraints. Both swing the answer across the whole spectrum.
- Define the question: a prompt needs to be as specific as you can make it, and the more effort you put into that the better your chances. This is the biggest lever you actually control. Draft it until you cannot meaningfully improve it by adding or specifying anything further.
- Define the goal and set the boundaries: what do you expect the model to give you, and what should it not do. Settling that here tells the model most of what you want out of the answer.
- Define answer length: short and quick, a long analysis, or something between. There is no objective distinction, it is a judgment call, so stating it up front settles it from your side and gives you a better chance at the outcome you had in mind. How the model behaves still depends on other factors.
- Define uncertainty: tell the model to ask questions and to assume nothing. This alone can save you from going down an entirely wrong direction and burning time and usage on it.
- Define your position: what do you concretely know, and are you a beginner, intermediate, or expert on the subject. This heavily alters the shape and depth of the response.
- Give examples: the best way to get something shaped the way you want. The closer an example sits to your actual case, the more accurately the model can map what you are asking for.
- Model selection: better model, better answer, though most frontier models converge on most questions and differentiate mainly on specific tasks, particularly agentic coding. As a rule of thumb, simpler question to a simpler and faster model, hard or ambiguous question to a more capable one.
- Effort level: This is becoming increasingly important as models are becoming increasingly capable at the tasks they are actually put to, and more often than not medium or even low effort is more than adequate, producing better consistency, speed, conciseness, and scope. This is where the document comes in, stating explicitly some of the behaviour that should be common across answers. Outside agentic coding and specific cases, a baseline frontier model more than suffices and is usually the value, cost, and speed effective option.
- Limit your expectations: the mechanism is probabilistic. By design it is extremely good at deterministic problems and less so at others. It WILL make mistakes, it will not be perfect, and you WILL have to guide it. Do not judge the model, or a change to your prompt document, by a failure to produce the exact thing you wanted when that thing was never defined.

## Background

Nothing here was designed up front. Every rule is a fossil of a specific observed failure, added after the failure was seen more than once. That provenance is the only evidence the rules aren't arbitrary, so it is recorded rather than smoothed away.

| Rule | The failure it was written against |
|---|---|
| Origin | A truth-priority prompt that produced compliance-shaped output. Diagnosis: abstract directives are superficially satisfiable, behavioral ones aren't. Yielded the first four: challenge the framing, flag comfort/truth divergence, steelman, detect confirmation-seeking |
| `PREMISE` | Insufficient premise scrutiny, confirmed across four separate threads and localized: it fires when the model engages sympathetically with the user's reasoning rather than leading analytically |
| `SYMMETRY` | Validation that survived being challenged only until it was challenged directly; and the inverse, correction asserted to look rigorous |
| `BASIS` / `UNKNOWN` | Fluent reconstruction from adjacent material, and confidence figures computed from nothing |
| `COMPRESS` | Verbosity drift flagged in thread after thread, with thread length itself identified as the re-trigger |
| `SIGNATURE` | Em-dash overuse, generalized on correction: the specific tell was the symptom, any default device is the disease |
| `[CODE]` | Extracted from a real build — visibility, fault tolerance, non-destructive defaults, config over hardcoding, no decorative console output |
| Cross-turn check | An earlier count-based version of the same rule, discarded because firing on turn count incentivizes manufactured dissent. Rewritten to fire on lapsed scrutiny |
| `UNDERSPEC` | Gap-filling from priors. Repeatedly narrowed, then widened to an unconditional ask default once it became clear the model's bias runs hard the other way |
| `AMENDMENT` | Rule saturation — the document growing by restating directives that already existed |

Structural revisions along the way: the tier system was re-cut from difficulty to verification method, six overlapping accuracy rules were collapsed into two, a `!command` / `--flag` layer replaced an earlier `Ignore Response Philosophy: N` override syntax.

## License

MIT — see [LICENSE](LICENSE). Copyright (c) 2026 royverd.
