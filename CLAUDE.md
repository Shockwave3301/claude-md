# CLAUDE.md

# General  
## Project Description
{{ INSERT THE GENERAL DESCRIPTION OF THE PROJECT AND ITS PURPOSE HERE }}

## Tech Stack
{{ INSERT THE TECH STACK TABLE HERE }}

## Project Structure 
{{ INSERT HIGH-LEVEL SCHEMA OF THE PROJECT STRUCTURE HERE }}

## Commands
{{ INSERT THE COMMANDS HERE }}

# Code Style
## Main Principle

Code is read far more often than it is written. Optimize every change for the next reader, who can hold about 4 facts in working memory at once. Cut *extraneous* load (how the code is written); the *intrinsic* difficulty of the problem can't be removed. Familiar is not the same as simple.

## Before writing

Go down this ladder and stop at the first "yes":

1. **Does this need to exist at all?** The cheapest feature is the one not built. If an edge case carries disproportionate complexity, propose dropping it instead of silently building it.
2. **Does the codebase already do it?** Reuse it (see *Reuse and duplication*).
3. **Does the platform, the standard library or an existing dependency already do it?** Use that.
4. Only then write new code, in as few files as possible.


## Scope of changes

- Change and create as few files as possible. Before adding a file, check whether the file that already owns the concern can take the change: a few lines there, a new parameter/prop on an existing function or component, an existing styling option instead of new styles.
- A small diff never outranks the reuse rules or the structure rules below.
- Don't refactor, reformat or rename outside the task. Mention opportunities in your answer instead.
- Delete temporary files your tools created in the working tree (screenshots, snapshots, logs) before finishing.

## Reuse and duplication

- If a function, hook, type or component already exists, reuse it, even across feature boundaries. Never write "your own version" of something that exists. Duplicate only when explicitly asked in the current task.
- If reuse needs a cross-feature import or a move into shared code, pick the lightest option that solves the task and mention the alternative in your answer.
- Don't *create* shared abstractions speculatively. Code that merely looks similar stays separate. Extract only when a second real caller needs the same logic for the same reason.
- Don't add a dependency for something you can write in ~20 lines. Every dependency is code we have to debug. When one is justified, pin the exact version (no `^`/`~`) and say why it's needed.

## Readability

- **Clarity over brevity.** Shrink the *amount* of code, never the *names*.
  ```ts
  // Good
  const filterActiveRecords = (records: Item[]): Item[] => records.filter((record) => record.isActive);
  // Bad
  const f = (d: any[]) => d.filter((i) => !!i.a);
  ```
- **Functions are named with verbs:** `getOrders`, `filterActiveRecords`, `navigateToDefaultTab`. Not `orders`, `activeRecords`.
- **Booleans use the `is` prefix,** even when it's grammatically awkward: `isPending`, `isError`. Not `pending`, `hasError`.
- **Functions are `const` arrow expressions,** not `function` declarations. This applies to all of them: utilities, handlers, hooks, components.
- **Conditions with more than 2 clauses:** pull the parts into named booleans, then combine those.
  ```ts
  const isExpired = expiresAt < now;
  const isAllowed = role === 'admin' || isOwner;
  if (!isExpired && isAllowed) { ... }
  ```
- **Guard clauses and early returns** for preconditions. Keep the happy path at the lowest indentation level. No more than 2 levels of nesting in new code.
- Use a minimal subset of language features. No clever tricks, type gymnastics or metaprogramming when plain code works. Pick the most boring idiom the codebase already uses.
- Plain words over jargon in names ("login"/"permissions" over "authn"/"authz").

## Comments

- Write comments in the language the codebase already uses.
- Explain **why** (motivation, constraints, non-obvious trade-offs), never **what** the next line does. A short overview at the top of a complex module is fine; narrating each line is not.
- Every deliberate workaround (manual optimization, lint suppression, opt-out directive) carries a comment explaining why it exists. Existing ones are documented exceptions: don't "clean them up". Before removing one, confirm with the linter or other tooling that it is really no longer needed.

## Types and data

- Keep data in its natural type. Convert only at the boundary that needs another type.
  ```ts
  // Good: the id stays a number, converted only where the input needs a string
  const [chainId, setChainId] = useState<number | null>(null);
  <Select value={chainId !== null ? String(chainId) : ''} onChange={(value) => setChainId(Number(value))} />
  // Bad: stored as a string, converted at every comparison and submit
  ```
- Avoid type assertions. Where one is unavoidable (parsing a network response, for example), add a `// SAFETY: ...` comment explaining why it holds.
- If modules have a dedicated types file, all of that module's types go there, even ones used in only one place.
- Use self-describing string values for statuses, enums and error codes (`"token_expired"`, not `3`). No numeric codes that need a lookup table in someone's head.
- Branch on machine-readable error codes, never on message text. Business errors are codes in the response body, not custom meanings for HTTP statuses.
- **Single source of truth.** Derive state instead of mirroring it (the active navigation item comes from the URL, not from separate state). Policies such as permissions or feature access are defined in exactly one place and consumed everywhere.

## Structure

- **Deep modules:** simple interface, substantial logic behind it. Create a new function, file, component or hook only if its interface is simpler than inlining its body. No `Factory`/`Manager`/`Provider`/repository wrappers whose name is harder to understand than their body.
- Don't split code into tiny functions to satisfy a line count. Linear code that reads top to bottom beats a chain of jumps between 5-line helpers.
- Composition over inheritance. Never add a level to an inheritance chain.
- No abstraction layers for architectural purity. Add one only for a concrete, current need: a second real implementation, a test seam for core logic, a real extension point.
- **Group by feature (domain), not by technical type.** Entry points (routes, pages, handlers) are thin shells; logic lives in features. Each feature exposes a small public API through its index file.
- **Placement follows who imports it.** Used by one consumer → colocated with that consumer. Used by several → shared. When a new importer appears, move it up.
- **kebab-case** for all file and directory names. Inside a feature folder, don't repeat the feature name (`api.ts`, not `orders-api.ts`).
- Keep business logic independent of the framework. Framework code calls into plain modules, not the other way round.
- Default to the simplest architecture that works: a monolith with well-isolated modules, standard CRUD. Defer irreversible structural decisions until requirements force them.
- Don't introduce new project-specific patterns, conventions or mental models without explicit approval.

## UI

- Use the project's design system first. Component props before custom CSS; custom styles only for what props can't express.
- Never hand-draw icons (inline SVG) or reimplement components the design system already provides. Use the project's icon set.
- Styles reference semantic tokens only, never raw palette values or hard-coded colors.
- Every data view handles loading, error (with retry) and empty states.
- Solve cross-cutting concerns (theming, responsive behavior, auth guards) once, centrally. Don't add per-screen overrides.

## External contracts

- API specifications and other files owned by other teams (OpenAPI/Swagger, generated code, vendored code) are read-only. If a requirement diverges from the contract, implement the requirement and report the divergence.
- All network requests go through the project's single API client.

## Definition of done

Every feature and non-trivial edit goes through these steps before it counts as done. Skip them only for trivial edits (copy change, rename, version bump).

1. **Review your own diff for what to delete:** unused code, unneeded files and abstractions, defensive branches for impossible cases.
2. **Check complexity** of every function that gained real branching. Cyclomatic complexity: 1–5 fine; 6–10 refactor if you're already in there; 11–15 refactor now; over 15 must be split.
3. **Run the project's lint, type check and build** (and tests, if the project has them). Fix what they report.
4. **Self-check:**
   - Can a newcomer understand this diff without opening more than ~3 other files?
   - Would debugging a failure here mean tracing through several indirections? If so, flatten.
   - Did I use a pattern or language feature the codebase doesn't already use? If so, justify it or replace it.

Review tools and skills are advisory. Rejecting a finding on purpose is fine; leaving it unexamined is not.

## Reporting

In your final answer, mention: divergences from external contracts, alternatives you didn't take (such as moving code into shared), review findings you rejected and why, and anything you couldn't verify.
