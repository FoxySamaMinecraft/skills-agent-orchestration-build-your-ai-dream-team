# Project Pulse implementation plan

## Summary

Build a small, static Project Pulse dashboard that lets contributors scan project
names, summaries, owners, statuses, recent activity, and priority or risk. Keep
the app dependency-free: serve files from `app/` over HTTP, load project records
from JSON, and render accessible, responsive project cards.

The repository is an exercise template. Its `.github/project-pulse-brief.md`,
agent definitions, step prompts, and `scripts/validate-exercise.sh` establish
the product requirements and conventions. There is no existing application or
frontend build/test setup to extend. Do not add a framework or package
dependencies for this static app.

## Ownership and deliverables

| Owner | File(s) | Responsibility |
| --- | --- | --- |
| Planner | `docs/project-pulse-plan.md` | Capture requirements, interface/data contract, sequencing, ownership, risks, and validation. |
| Designer | `app/styles.css` | Define visual hierarchy and responsive, accessible styling; coordinate selectors and component states with Coder before implementation. |
| Coder | `app/index.html` | Create the semantic page structure and client-side rendering/error behavior; link the stylesheet and load the data. |
| Coder | `app/project-data.json` | Supply representative, valid project records matching the agreed schema. |
| Coder | `.vscode/launch.json` | Add the strict-JSON VS Code launch configuration that serves `app/` and opens `index.html`. |
| Orchestrator | All deliverables at integration | Assign scopes, resolve interface questions, review the combined result, run/report validation; avoid editing specialist-owned files unless coordinating a correction with that owner. |

## Interface and data contract

Agree on this contract before parallel implementation:

- The page title and visible heading are **Project Pulse**. Use semantic page
  landmarks and a `.dashboard` root container.
- Render one `.project-card` per item in a top-level `projects` JSON array.
- Every project has `name`, `summary`, `owner`, `status`, `recentActivity`, and
  `priority`. Values are plain displayable strings; use representative,
  contributor-friendly content and consistent status/priority labels.
- Display all six fields, including the contributor-friendly summary. Status
  and priority must remain understandable in text, not color alone.
- The HTML owns rendering and loading states; CSS owns presentation and
  responsive behavior. Agree on any additional hooks needed for the page
  heading, status/priority badges, loading indicator, and error/empty messages.
- Fetch JSON via HTTP from `project-data.json` relative to the page. A failed
  request, invalid payload, or empty projects array must produce a visible,
  useful message rather than a blank page. Keep the static HTML usable while
  loading and expose the completion/error state accessibly.
- The `.vscode/launch.json` configuration is named **Run Project Pulse
  Dashboard**, runs `python3 -m http.server 5500` with working directory
  `${workspaceFolder}/app`, and opens
  `http://localhost:%s/index.html` through `serverReadyAction`. Use a VS Code
  launch type that supports running that command in a terminal and the
  `serverReadyAction`; do not launch the server root as the browser target.

## Ordered work streams

### 1. Confirm requirements and agree the contract

- **Owner:** Orchestrator, with Planner as the requirements reference.
- **Inputs:** `.github/project-pulse-brief.md`, this plan, and agent scopes in
  `.github/agents/`.
- **Output:** Agreement on the selectors, data fields, status/priority values,
  loading/error states, and launch behavior above.
- Resolve any disagreement before implementation. In particular, keep the
  brief's required fields and include its short summary requirement in the data
  and UI.

### 2. Implement presentation and application in parallel

Start only after step 1 has fixed the contract. Assign disjoint file scopes:

- **Designer — `app/styles.css`:** Create the polished visual system, readable
  typography and spacing, project-card layout, status/priority treatments, and
  narrow-screen adaptation. Include clear focus indicators, sufficient
  contrast, and non-color status cues. Coordinate selectors/state classes with
  Coder, but do not edit the HTML or data.
- **Coder — `app/index.html`:** Build the semantic dashboard shell, loading
  status, and cards renderer. Link `styles.css`, fetch `project-data.json`,
  display every contracted field, and handle fetch/parse/schema/empty-data
  failures with a visible message. Use accessible labels/landmarks and
  appropriate live status semantics for asynchronous updates. Do not embed the
  project data in markup as a substitute for loading the JSON file.
- **Coder — `app/project-data.json`:** Create several realistic entries with
  all six contracted fields; ensure consistent, clear status and priority
  values. Keep JSON strict (no comments or trailing commas).
- **Coder — `.vscode/launch.json`:** Add the named launch configuration with
  strict JSON, `cwd` set to `${workspaceFolder}/app`, the specified Python HTTP
  server command on port `5500`, and `serverReadyAction` that opens
  `/index.html` using the detected port. This file is independent of the UI
  implementation aside from the agreed port and URL.

### 3. Integrate and validate sequentially

- **Owner:** Orchestrator coordinates; Designer and Coder correct issues within
  their assigned files.
- Wait until both implementation scopes are complete. Check the real data,
  markup, selectors, visual states, server command, and browser URL together.
- Run the checks below, exercise the launch flow, and have owners fix any
  failures before reporting completion.

## Dependencies and parallel-work decisions

- The interface/data contract in step 1 is a prerequisite for both
  implementation scopes; neither specialist should invent incompatible field
  names or selectors.
- Once the contract is agreed, Designer's CSS and Coder's HTML/data/config work
  can proceed in parallel because their file ownership does not overlap.
- CSS and HTML have a **contract dependency**: Coder should share planned hooks
  and loading/error states with Designer before finishing markup; Designer
  should style those agreed states. This coordination does not require either
  to edit the other's files.
- HTML behavior depends on the JSON field names and expected states. Coder can
  define the data schema and renderer together; confirm the shared schema
  before either is considered complete.
- The launch file can be prepared in parallel after the agreed port/path. Its
  final smoke test depends on the completed `app/index.html` and JSON.
- Integration and browser verification are sequential and must follow all
  implementation work.
- No external runtime or build dependencies are expected. If a browser or VS
  Code launch environment is unavailable, perform the deterministic static and
  HTTP checks and report the unavailable manual check explicitly.

## Risks and edge cases

- A browser may block `fetch()` for a page opened directly using `file://`; use
  the launch configuration's local HTTP server.
- Validate the fetched payload before rendering so malformed JSON, a missing or
  non-array `projects` property, and an empty array do not result in a blank or
  broken interface.
- Keep error/loading messaging visible to assistive technology; do not rely on
  color or icons alone to communicate status or priority.
- Narrow screens and zoom can cause long names or activity text to overflow;
  cards must wrap/reflow without horizontal page scrolling.
- Port `5500` may be in use. If it must change, update both the server command
  and the browser URL pattern consistently, then repeat launch validation.
- `serverReadyAction` must open `index.html`, not the server's directory
  listing.

## Validation and acceptance

Run these checks from the repository root after implementation:

1. **JSON syntax and contract**
   - `python3 -m json.tool app/project-data.json` and
     `python3 -m json.tool .vscode/launch.json` both exit successfully.
   - Inspect the project data: top-level `projects` is a non-empty array and
     each record contains non-empty `name`, `summary`, `owner`, `status`,
     `recentActivity`, and `priority` fields.
2. **HTML and rendering behavior**
   - Inspect `app/index.html` for title/heading **Project Pulse**, semantic
     landmarks, a `.dashboard` root, a stylesheet link to `styles.css`, and a
     request for `project-data.json`.
   - Serve from `app/` and verify in a browser that each data record yields
     exactly one visible `.project-card` and that all six fields appear in the
     matching card.
   - Exercise successful load, empty project list, invalid payload, and failed
     request; confirm loading resolves and each failure has a useful visible
     message instead of an empty/uncaught page.
3. **CSS, responsive, and accessibility behavior**
   - Inspect `app/styles.css` for `.dashboard` and `.project-card` rules,
     intentional typography/spacing, clearly distinguishable status and
     priority cues, and narrow-screen responsive rules.
   - At desktop and narrow/mobile viewport widths, verify cards reflow, text
     wraps, no content is clipped, and the page has no unintended horizontal
     scrolling. Also check at increased zoom.
   - Navigate using the keyboard and verify visible focus, logical reading and
     focus order, semantic headings/landmarks, and accessible loading/error
     announcements. Confirm status and priority labels remain clear without
     color. Check text/background contrast using browser accessibility tools
     or an equivalent contrast checker.
4. **Launch configuration and HTTP smoke test**
   - Inspect `.vscode/launch.json` for strict JSON, exact configuration name,
     launch type/command support, `cwd` `${workspaceFolder}/app`, command
     `python3 -m http.server 5500`, and `serverReadyAction` target
     `http://localhost:%s/index.html`.
   - Start **Run Project Pulse Dashboard** from VS Code Run and Debug. Verify
     the browser opens the dashboard (not a directory listing), and confirm the
     page and `project-data.json` are served successfully. Stop the server
     after the smoke test.
   - If VS Code is unavailable, start the same HTTP server from `app/` and
     request both `/index.html` and `/project-data.json`; record that this is a
     server smoke test, not a launch-configuration verification.

The repository's `scripts/validate-exercise.sh` checks exercise/template
conventions and launch-file presence/JSON, but does not exercise browser
rendering, responsive layout, accessibility, or the VS Code launch itself.
Run it when validating the overall exercise, in addition to the checks above.
