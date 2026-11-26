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
