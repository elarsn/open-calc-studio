# Open Calc Studio, Stadsverkstad fork

Fork of OpenAEC-Foundation/open-calc-studio that adds embodied carbon next to cost.
Owner: Erik Larsen, Stadsverkstad AB. He writes in Swedish; answer him in Swedish.

## Read first

- `docs/stadsverkstad/UTVECKLINGSPLAN.md`: the development plan (three steps).
- `docs/research/calculation-packages-2026-09-28/issues/07-embodied-carbon.md`: upstream's own proposal for embodied carbon. Step 1 follows its scope and acceptance criteria.

## Branches

- `main` mirrors upstream. Never commit to it.
- `stadsverkstad` is the integration branch. Branch `feat/<name>` from it and open pull requests against it, in this fork (elarsn/open-calc-studio), never against upstream unless Erik asks.

## Commands

- `npm ci` (clone with `--recurse-submodules`)
- `npx tsc --noEmit`
- `npm run test:local` (about 2 000 tests, a few minutes)
- `npm run dev` serves on http://localhost:4200

## Rules for this fork

- Monetary totals must never change because of the carbon module. The existing test suite must stay green untouched.
- A missing carbon factor is incomplete coverage, never zero.
- Keep changes additive: new files and optional fields, minimal edits to existing files. Upstream releases weekly and this fork has to merge it.
- Old `.ifcCalc` files must open and save unchanged.
- Never call the result a certified LCA or a climate declaration.
- The repo is public: no licensed price lists, client data or project data.
- Code and comments in English. UI strings through i18n (`src/i18n/locales`), at least `en`, `sv`, `nl`.
- Existing domain terms are Dutch (`regel` = resource line, `staart` = markups at the foot of the estimate, `hoofdstuk` = chapter, `materiaal`/`arbeid`/`materieel` = material/labour/equipment). Keep them as they are.

## Working method

Erik wants this project run as an orchestrated loop. He has asked for subagents explicitly, so use them.

### Roles

| Role | Model and effort | Does | Does not |
|---|---|---|---|
| Orchestrator (the main session) | Fable, high effort | Reads the code, owns the plan and the data model, cuts the work into packages, reviews functionality, decides when a package is done, reports to Erik | Write feature code |
| Implementer | Opus 5.5, medium effort, one subagent per work package | Writes the code and tests for one package, iterates on findings | Change scope, touch files outside the package without asking the orchestrator |
| Design agent | Subagent that loads the `frontend-design:frontend-design` skill before doing anything else | Writes the UI specification before any UI code, reviews the built UI from screenshots | Write production code |
| Reviewer | Fresh subagent with no view of how the code was produced | Reads the diff against the package's acceptance criteria and the rules above, reports findings | Fix what it finds |

The orchestrator's own model and effort are set by Erik when he starts the session. If the session is not running Fable at high effort, say so before starting.

### Work packages

Each package the orchestrator hands out states: the goal and what it is for, the files and types involved, the acceptance criteria as checks that can be run, the constraints from this file that apply, and what to return (what changed, what was run and its result, what is uncertain). A subagent starts with no context, so the brief has to stand on its own.

Packages that do not touch the same files run in parallel, each in its own git worktree. The data model comes first and alone, because everything else depends on it.

### The loop for one package

1. The implementer builds the package and runs `npx tsc --noEmit` and the tests it touched.
2. Gates, run by the orchestrator, not taken from the implementer's report: type check, full test suite, monetary totals unchanged.
3. The reviewer reads the diff. For UI packages the design agent also reviews screenshots of the running app.
4. The orchestrator checks the functionality itself by using the feature in the running app (`npm run dev`), including the awkward cases: missing factor, unit mismatch, variant switch, an old file.
5. Findings go back to the same implementer, which keeps its context. Repeat from step 1.
6. A package is done when the gates pass, the reviewer has no open findings that matter, and the orchestrator has seen the feature work. After three rounds without convergence, stop and bring the question to Erik instead of looping on.

### UI work

- The design agent goes first: it studies the existing interface (Blueprint styling, the grid, the ribbon, the panels) and writes a short UI specification for the package. The carbon features should look like they were always part of the program, not like an add-on.
- The implementer builds to that specification.
- The design agent reviews the result from screenshots at desktop and mobile widths, in light and dark where the app supports it. Its findings go through the same loop.

### What reaches Erik

- The data model proposal, for approval, before any implementation package is handed out.
- The UI specification, for approval, before UI packages are handed out.
- At the end of each step: what was built, what was verified and how, what is uncertain, and which existing upstream files were edited.
- Anything that would change the plan, the scope or the rules in this file.
