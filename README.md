# Drive.NET

Drive.NET is a Windows desktop automation toolkit built for agentic AI. It exposes Windows desktop applications through Model Context Protocol (MCP), so AI agents — such as GitHub Copilot, Claude, or any MCP-compatible client — can discover processes, inspect UI trees, interact with controls, capture visual evidence, run accessibility analysis, and execute deterministic multi-step UI workflows.

Drive.NET works with any application that exposes a Microsoft UI Automation surface, including applications built with WinForms, WinUI 3, WPF, UWP, Electron, Flutter, Qt, Java/Swing, and Win32/MFC. It operates entirely externally — no code injection, no pixel matching, and no modification of the target application.

While Drive.NET can be used manually through its CLI and Companion app, it is ideally driven by an agentic AI that can plan, observe, and react to a live desktop application in real time. The MCP server gives the agent a structured, tool-based interface to the full Windows UI Automation surface.

## Choose Your Surface

| If you need to... | Use this surface | Why |
|---|---|---|
| Let GitHub Copilot or another MCP client explore a live app iteratively | MCP Server | Best for agent-driven workflows where the next action depends on what the agent discovers. |
| Run one deterministic command or a scripted step from a terminal or CI | CLI | Best for one-shot discovery, inspection, reporting, capture, lifecycle control, and test execution. |
| Visually inspect a live UIA tree, read findings, and triage accessibility issues with direct feedback | Companion | Best for onboarding, manual analysis, snapshot comparison, pattern inspection, and interactive debugging. |
| Manage workspaces, check server health, and apply updates | Helper | Best for installation management, workspace configuration, and keeping Drive.NET up to date. |

Typical flow for a new user:

1. Run the installer and use **Helper** to configure your workspaces.
2. Use **Companion** to understand the target app and inspect its live UIA surface.
3. Use the **CLI** when you want repeatable one-off commands or artifact generation.
4. Use the **MCP server** when you want an agent to drive the app end to end.

---

## Features

### Process Discovery & Session Management

- Find running desktop processes with visible windows, optional window inventories, and parent/child process hierarchies
- Filter by process name, .NET-only, or application-specific branches (e.g. Firefox, Electron)
- Attach to a target application through a reusable session; retarget to secondary windows, dialogs, or flyouts as workflow demands
- Session-start warning toast with acrylic styling and a slide-in countdown before automation begins (configurable, per-session overridable)
- Window handle aliases for quick retargeting between named windows
- Connect filters: `processName`, `processId`, `windowTitleRegex`, `executablePath`, `commandLineContains`
- `connectNewest` / `connectLatest` actions for multi-instance targets
- Session groups for coordinating multiple connected applications with named roles

### Element Querying & Inspection

- Find UI elements by automation ID, name, control type, class name, or hierarchical path selectors (e.g. `Pane[automationId=MainPanel] > Button[name=Save]`)
- Enumerate the element tree with configurable depth (0–25) and node budgets (1–5,000)
- In-band tree truncation with continuation hints for incremental subtree drilling
- Read element properties, children, parents, and grid/table data
- `matchIndex` for disambiguating repeated sibling elements
- Verbose detail mode with class names, help text, patterns, accelerator keys, access keys, item status, labeled-by references
- Exhaustive single-element inspection: all UIA properties, supported patterns, available actions, current value, toggle/expand state
- Framework artifact annotation for WinUI `TitleBar` placeholders and similar provider artefacts
- Scoped searching: `descendants`, `children`, or `subtree`
- Selector explain: verify how a selector resolves before acting, showing the exact resolution path without side effects

### UI Interaction

- **Click actions** — click, double-click, right-click with returned target element confirmation
- **Text input** — type (clears then types), clear, sendKeys (keyboard shortcuts like `Ctrl+S`, `Alt+F4`)
- **Selection** — select combo box / list items by name
- **Toggle & expand** — toggle checkboxes, expand/collapse tree nodes and collapsible regions
- **Scroll** — scrollIntoView to bring offscreen elements into the viewport before interaction
- **Drag & drop** — drag element to a target element, or smooth mouse movement with human-like easing and optional mouse button hold
- **Focus & highlight** — set keyboard focus, flash/highlight elements for visual confirmation before critical actions
- **Clipboard** — read and write system clipboard content
- **Window-level input** — `sendKeys` and `type` without an `elementId` send keystrokes directly to the foreground window, useful for browser content areas and surfaces without UIA elements
- **Window handle targeting** — optionally target a specific window for keystroke delivery
- **Secure text entry** — registered secrets can be typed without being echoed in responses or logged

### Condition Waiting

- Wait for element existence, removal, property changes, text equality, enabled state, visible state, window open/close, and structure changes
- Polling-based detection (250ms interval) and UIA event-driven detection for structure changes
- Configurable timeouts up to 300 seconds
- Selector explain mode for debugging wait conditions before committing the timeout budget
- Hierarchical path selectors and `matchIndex` for precise condition targeting

### Window Management

- List all visible top-level windows with position, size, visibility, and modal state
- Detect blocking modal dialogs with ranked suggested actions and confidence levels
- Dismiss blockers with one-shot recovery — click recommended or named buttons
- Resize, move, minimize, maximize, restore, close, bring to front
- Browser onboarding and first-run surface classification via automation branching
- Session retargeting to move the automation root to a different top-level window

### Desktop Metrics

- Query connected monitors with bounds, working area, and DPI scale
- Identify the current foreground window

### Screenshot Capture

- Capture entire windows or individual elements
- Base64 inline MCP images for direct AI agent viewing
- File output as PNG for durable evidence
- Window handle targeting for secondary windows

### Batch Automation

- Execute up to 100 `query`, `interact`, `wait_for`, and `assert` steps atomically in a single tool call
- Variable binding: `saveAs` saves the first matched element ID, `save` extracts result fields via JSON path, `${name}` references in later steps
- Conditional execution: `when` with variable equality, inequality, and existence checks
- Per-step retry policies: `maxAttempts`, `delayMs`, `backoffMs`
- Timing control: `delayBetweenMs` at the batch level, `delayBeforeMs` / `delayAfterMs` per step
- Per-step error handling: `continueOnError` overrides batch-level `stopOnError`
- Per-step session overrides for cross-application workflows within a single batch call
- Reserved execution variables: `lastStepSuccess`, `lastStepSkipped`, `lastStepError`, `step1Success`, etc.
- Selector explain steps for in-batch diagnostics

### Declarative Assertions

- Evaluate UI element state, properties, and element counts without writing procedural code
- Composable clauses: property equals, contains, exists, enabled, visible, count comparisons
- Available as a standalone MCP tool, CLI command, and within batch steps

### Live Observability

- Subscribe to continuous UI change events: structure changes, property changes, focus changes, and invocations
- Drain queued events on demand for agent-driven reactive workflows
- Event-driven alternative to polling for monitoring dynamic UI state

### Automation Analysis & Reporting

21 checks across Automation Readiness and Accessibility Compliance:

| Code | Check | Severity |
|---|---|---|
| DNC001 | Missing AutomationId on interactive elements | Error |
| DNC002 | Missing or empty Name property | Error |
| DNC003 | Missing LabeledBy on input controls | Warning |
| DNC004 | Duplicate AutomationIds within same parent | Error |
| DNC005 | Missing keyboard access (AcceleratorKey / AccessKey) | Warning |
| DNC006 | Empty clickable region (zero-size bounding rectangle) | Critical |
| DNC007 | No supported UIA patterns on interactive elements | Error |
| DNC008 | Deep nesting beyond configurable threshold | Info |
| DNC009 | Read-only ValuePattern elements without visual indicator | Hint |
| DNC010 | Missing HelpText on complex controls | Info |
| DNC011 | Offscreen element with visible bounds | Warning |
| DNC012 | Focus order not matching visual layout order | Warning |
| DNC013 | Missing ToolTip on icon-only buttons | Warning |
| DNC014 | Landmark/GroupBox without accessible name | Info |
| DNC015 | Live region without UIA notification support | Hint |
| DNC016 | Overlapping interactive elements | Warning |
| DNC017 | Inconsistent sibling naming | Warning |
| DNC018 | Ambiguous element selector (not uniquely targetable) | Warning |
| DNC019 | Unexpected LabeledBy on static content | Warning |
| DNC020 | Input name mirrors current value instead of field purpose | Warning |
| DNC021 | Large framework surface lacks semantic descendants | Warning |

- Export as Markdown, JSON, HTML, or SARIF v2.1.0
- Remediation plan generation with prioritized, framework-specific fix guidance
- Reports and plans are written under the workspace `.drive-net` directory
- Framework detection: WinForms, WinUI 3, WPF, UWP, Flutter, Electron, Qt, Java/Swing, Win32/MFC
- Score tiers: Gold, Green, Amber, Red, or Inconclusive (when UIA surface is incomplete)
- Configurable severity overrides per check

### Snapshot Comparison

- Create `.dncsnap` analysis baselines from live sessions
- Compare two snapshots to track improvement or detect regressions
- Diff output in Markdown or JSON
- Artifacts written under the workspace `.drive-net` directory

### Record-to-Script

- Record live interactions, queries, and waits into replayable scripts
- Output as batch JSON (for `batch` tool) or YAML test suites (for `test` command)
- Automatic `saveAs` / variable insertion when elements are queried then interacted with
- Duplicate consecutive wait coalescing to reduce noise
- Secret value redaction: registered secrets are replaced with `[REDACTED]`
- Available via MCP `record` tool, CLI `record` command, and Companion interaction tester

### Application Lifecycle

- Launch target applications with configurable arguments and working directory
- Stop running applications gracefully or forcefully
- Query application status (running, PID, window title)

### Application Branching

- Automation branching isolates application-specific behaviour from the generic UI automation pipeline
- **Firefox branch**: auto-enriches discovery, supports newest-instance attach filtering, deprioritizes onboarding windows, classifies and dismisses first-run blocker surfaces
- **Electron branch**: auto-enriches discovery for Electron-based desktop apps, classifies DevTools windows as secondary surfaces, and adapts to the single-content-pane UIA pattern common in Chromium-hosted apps
- **Win32 branch**: provides awareness of common Win32/MFC dialog classes, deprioritizes common dialogs (Open, Save As) during window resolution, and adjusts blocker classification for legacy desktop applications

---

## Included Components

| Component | Description |
|---|---|
| **MCP Server** | 17 tools, 4 resources, and 3 prompts over stdio transport |
| **CLI** | 21 one-shot commands for discovery, inspection, interaction, analysis, testing, and more |
| **Companion** | WinUI 3 desktop app for interactive analysis, element exploration, code generation, recording, and snapshot comparison |
| **Helper** | WinUI 3 workspace manager for adding workspaces, checking health, viewing server status, and applying updates |
| **Analysis Engine** | Shared analysis, report, and snapshot tooling used by MCP, CLI, and Companion |
| **Copilot Skills** | 12 workspace-scoped skill files for targeted agent guidance |
| **Installer** | Install, update, uninstall, and artifact cleanup — also available as PowerShell scripts for automated workflows |

---

## MCP Server

### Tools

| Tool | Description | Read-Only |
|---|---|---|
| `discover` | List running processes, optional window inventories and process hierarchies | Yes |
| `session` | Connect to, retarget, or disconnect from a target application | No |
| `query` | Find/resolve elements, enumerate trees, read properties, grid data, explain selectors | Yes |
| `inspect` | Exhaustive element info: properties, supported patterns, available actions | Yes |
| `interact` | Click, type, select, toggle, expand, collapse, drag, mouseMove, sendKeys, clipboard, highlight | No |
| `wait_for` | Wait for UI conditions with polling or event-driven detection; explain selectors | Yes |
| `window` | List windows, detect/dismiss blockers, resize, move, minimize, maximize, restore, close | No |
| `desktop` | Query OS-level monitor bounds, DPI scale, and foreground window | Yes |
| `capture` | Screenshot windows or elements as base64 inline images or PNG files | No |
| `batch` | Execute up to 100 query/interact/wait_for/assert steps atomically with variable binding | No |
| `report` | Run automation analysis and write Markdown/JSON/HTML/SARIF/plan artifacts | No |
| `snapshot` | Create or compare reusable `.dncsnap` analysis baselines | No |
| `lifecycle` | Launch, stop, or query the status of a target application | No |
| `assert` | Evaluate declarative assertions against UI element state, properties, and counts | Yes |
| `secret` | Register, revoke, or clear in-memory secrets for secure data entry | No |
| `observe` | Subscribe to live UI events (structure, property, focus, invoke) and drain queued changes | Yes |
| `record` | Record interactions into replayable batch JSON or YAML test suites | No |

### Resources

| Resource | URI | Description |
|---|---|---|
| `sessions` | `drivenet://sessions` | Enumeration of active sessions with status |
| `session-windows` | `drivenet://session/{sessionId}/windows` | Top-level windows for a connected session |
| `session-element-tree` | `drivenet://session/{sessionId}/tree` | Live element tree for inspection-first diagnostics |
| `session-events` | `drivenet://session/{sessionId}/events` | Event subscription management URI for live observability |

### Prompts

| Prompt | Description |
|---|---|
| `drivenet-usage` | General usage guide: connect, query, interact, capture |
| `drivenet-testing` | UI testing workflows: find elements, assert state, verify |
| `drivenet-debugging` | Investigating UI issues: inspect elements, capture, explore tree |

---

## CLI

Fast one-shot commands that reuse the same core services as the MCP server. Each command opens its own short-lived session when it needs to attach to a process.

### Commands

| Command | Description |
|---|---|
| `doctor` | Check UI Automation access, report session type and OS details |
| `discover` | List visible-window processes with optional filters |
| `pick` | Resolve one deterministic process target and suggest an exact PID for later commands |
| `desktop` | Query monitors and foreground window |
| `windows` | List/manage windows: list, blockers, resize, move, minimize, maximize, restore, close, bringToFront |
| `find` | Find matching UI elements with flat or hierarchical selectors |
| `inspect` | Verbose single-element detail: patterns, value, toggle state |
| `tree` | Render UI Automation hierarchy with depth/node budgets and truncation hints |
| `interact` | Perform actions: click, doubleClick, rightClick, type, clear, sendKeys, select, toggle, expand, collapse, scrollIntoView, dragTo, mouseMove, setFocus, highlight, clipboard |
| `wait_for` | Wait for UI/window conditions with configurable timeout |
| `batch` | Execute multi-step JSON sequences atomically with variable binding |
| `playback` | Replay recorded batch JSON or YAML suite/manifest files from one CLI entry point |
| `capture` | Screenshot windows or elements to PNG |
| `demo` | Run built-in visual validation flows such as `demo mouse-move` |
| `report` | Run analysis pipeline and write report artifacts |
| `snapshot` | Create or compare analysis baselines |
| `test` | Run YAML test suites or manifests with structured result output |
| `lifecycle` | Launch, stop, or query the status of a target application |
| `assert` | Evaluate declarative assertions against UI element state |
| `observe` | Subscribe to live UI automation events and print them as they arrive |
| `record` | Record interactions into batch JSON or YAML test suites |

### Quick Examples

```powershell
DriveNet.Cli.exe doctor
DriveNet.Cli.exe discover --dotnet-only
DriveNet.Cli.exe pick --process-name MyApp --newest --json
DriveNet.Cli.exe desktop --action monitors
DriveNet.Cli.exe find --process-name MyApp --automation-id SubmitButton
DriveNet.Cli.exe find --process-name MyApp --path "Pane[automationId=MainPanel] > Button[name=Save]" --explain
DriveNet.Cli.exe interact --process-name MyApp --action click --automation-id SubmitButton
DriveNet.Cli.exe wait_for --process-name MyApp --condition elementExists --automation-id SuccessLabel
DriveNet.Cli.exe playback --input recorded.json --process-name MyApp
DriveNet.Cli.exe capture --process-name MyApp --automation-id SubmitButton
DriveNet.Cli.exe demo mouse-move --process-name MyApp
DriveNet.Cli.exe report --process-name MyApp --format markdown
DriveNet.Cli.exe snapshot create --process-name MyApp --output snapshots/latest.dncsnap
DriveNet.Cli.exe test --manifest tests/definitions/manifest.yaml --result-json results.json
```

All commands default to human-readable output and support `--json` for scripting and agentic consumption.

---

## YAML Test Runner

Deterministic, repeatable UI automation workflows authored in YAML:

- **Manifests** run multiple suites in one invocation
- **Suites** maintain one live session for ordered steps, so saved element IDs remain valid
- **App lifecycle**: launch executables, launch helper apps, `single_instance` policies (`reuse` / `restart`), configurable startup wait
- **Step tools**: `discover`, `query`, `interact`, `wait_for`, `window`, `capture`, `desktop`
- **Variables**: `saveAs` for element IDs, `save` with JSON path extraction, `${name}` references in later steps
- **Expectations**: `success`, `count`, `count_gt`, `count_gte`, `contains`, `not_contains`, `property`/`equals`, `exists`, `condition_met`, `file_exists`
- **Conditional execution**: `when` with variable equality, inequality, and existence checks
- **Retry policies**: `max_attempts`, `delayMs`, `backoffMs`
- **Timing**: `delay_before_ms`, `delay_after_ms`, `continue_on_failure`
- **Lifecycle blocks**: `setup`, `teardown`, `finally` / `cleanup` (always runs, even on setup failure)
- **Reserved variables**: `lastStepIndex`, `lastStepSuccess`, `lastStepSkipped`, `lastStepError`, and per-app variables (`appProcessId`, `appWindowTitle`, etc.)
- **Structured output**: `--result-json` for machine-readable per-step payloads, saved variables, and raw command errors for agentic triage
- **JUnit XML**: `--junit-xml` for CI integration

```yaml
name: Login Flow Tests
app:
  exe: path/to/MyApp.exe
  startup_wait_ms: 3000

tests:
  - name: fill and submit login form
    steps:
      - tool: query
        action: find
        args:
          automation_id: txtUsername
        save_as: username_field

      - tool: interact
        action: type
        args:
          element_id: ${username_field}
          value: "alice@example.com"

      - tool: interact
        action: click
        args:
          automation_id: btnLogin

      - tool: wait_for
        args:
          condition: elementExists
          automation_id: lblWelcome
        expect:
          success: true
```

---

## DriveNet Companion

A standalone WinUI 3 desktop application for interactive accessibility and automation-readiness analysis. Companion provides a visual workbench for exploring, analysing, and testing the UIA surface of any running Windows desktop application.

### Target Selection & Session

- Hierarchical process/window tree with parent-child grouping and application icons
- Per-window targeting with auto-selection when a filter narrows to one target
- Persistent session with retargeting to secondary windows and dialogs

### Analysis

- **21 analysis checks** across Automation Readiness and Accessibility Compliance
- **Score tiers**: Gold, Green, Amber, Red, and Inconclusive
- **Framework detection**: WinForms, WinUI 3, WPF, UWP, Flutter, Electron, Qt, Java/Swing, Win32/MFC — with framework-specific remediation guidance and severity adjustments
- **Analysis mode toggle**: switch between Automation Readiness and Accessibility Compliance modes
- **Severity filtering**: view findings by severity level
- **Watch mode**: periodic automated re-analysis with change detection and timeline logging
- **Configurable severity overrides** per check

### Remediation Planning

- Prioritized fix guidance generated from analysis findings
- Effort estimation (quick, moderate, complex) and impact severity ranking
- Framework-specific recommendations grouped by control type or pattern

### Export

- **Markdown** — human-readable findings with recommendations
- **JSON** — machine-parseable findings for tooling
- **HTML** — rich formatted report
- **SARIF v2.1.0** — for code-scanning and CI workflows
- **AI Prompt** — generated remediation guidance exportable to file or clipboard

### Interactive Tools

- **Tree Explorer** — lazy-loaded UI Automation element hierarchy with search, property viewing, configurable depth/node budgets, and framework artifact annotation
- **Element Picker** — crosshair selection tool with hover highlighting, element flash, and timeout-based auto-dismissal
- **Interaction Tester** — test element interactions (invoke, toggle, value, expand/collapse, selection, range value, scroll) with real-time feedback and action logging
- **Code Generation** — generate interaction, wait condition, and batch sequence code from picked elements; build multi-step form automation sequences
- **Pattern Coverage Map** — UIA pattern availability matrix across the element tree with coverage percentages and framework-aware recommendations
- **Snapshot Comparison** — create `.dncsnap` baselines and diff them to track score changes, new findings, and resolved issues over time

### Recording

- Record live interactions into replayable batch JSON or YAML test suites
- Automatic variable binding when elements are queried then interacted with
- Secret value redaction for registered secrets

### Element Highlighting

- Colour-coded overlays on flagged elements in the target application by severity (critical/error, warning, info)
- Configurable highlight duration with extended timing for picker selections

### UI

- Full keyboard navigation with configurable shortcuts (Ctrl+O, Ctrl+T, Ctrl+Shift+A, etc.)
- Context menus with copy-to-clipboard actions
- Dark/light/high-contrast theme support with acrylic backdrop
- Status bar with framework badge, connection status, and running state indicator

### Command-Line Options

- `--connect-self` — auto-connect to Companion's own window on launch
- `--connect-self-analysis` — connect and auto-run analysis (used for self-regression testing)
- `--open-analysis` — navigate directly to the Analysis page on startup
- `--allow-multiple-instances` — bypass single-instance enforcement

---

## DriveNet Helper

A standalone WinUI 3 desktop application for managing the Drive.NET installation, workspaces, and server health. Helper launches automatically after installation and is the recommended way to configure and maintain Drive.NET.

### Workspaces

- **Add Workspace** — pick any folder and configure it as a managed workspace
- **Drag-and-Drop** — drop folders onto the dashboard to register them as workspaces
- **Scan Root Directory** — recursively scan a parent folder to auto-discover and register workspace candidates
- **Workspace Dashboard** — all managed workspaces at a glance with health status indicators (healthy, needs refresh, broken)
- **Per-Workspace Actions**:
  - Open in preferred editor (VS Code, Rider, Cursor, Visual Studio, etc.)
  - View and edit `.vscode/mcp.json` with live JSON validation
  - Browse installed Copilot skills with Update All capability
  - Refresh individual workspace configuration
  - Remove from managed list without deleting files
- **Fix All** — batch-repair all unhealthy workspaces in one action

### Server

- Live server status dashboard with running/stopped indicator and process information (PID, uptime)
- Start and stop the DriveNet.Server process
- Server log viewer with auto-refresh and severity-level filtering (Trace through Fatal)
- Log file management — refresh, open in editor, or clear log files

### Doctor

- Run `doctor` diagnostics with formatted output showing environment summary, install root, version, managed workspace count, OS, and architecture

### Updates

- Check for new Drive.NET releases from GitHub
- Download and apply updates with progress and file verification
- Import pre-downloaded release bundles for offline updates
- View update history with version, date, and source (online/offline)

### Tools

- Launch DriveNet Companion directly from the Helper
- View installation paths for all Drive.NET components

### Settings

- Theme selection (System, Light, Dark) with immediate UI update
- Preferred editor command configuration
- Automatic update checking on startup (toggle)
- Desktop shortcut creation for Doctor, Discover, and Helper
- Installation path display with clickable folder links

### Uninstall

- Clean removal of Drive.NET installation with optional workspace configuration cleanup

---

## Designed for Agentic AI

Drive.NET is purpose-built so an AI agent can autonomously operate a desktop application:

- **Structured tool interface** — every operation is a typed MCP tool call with validated inputs and structured JSON responses, not free-form screen scraping
- **In-band tree truncation** — large UI trees are bounded by a configurable node budget and return continuation hints so the agent can drill into subtrees without oversized payloads
- **Selector explain** — the agent can verify a selector resolves correctly before committing to a click or wait, reducing wasted actions
- **Blocker detection** — `window blockers` detects modal dialogs and returns ranked suggested actions (with confidence levels) so the agent can unblock itself
- **Variable binding in batch** — multi-step workflows can save element IDs and result fields into named variables, enabling data-dependent automation in a single tool call
- **Copilot skills** — 12 workspace-scoped skills provide domain-specific guidance for discovery, querying, interaction, waiting, capture, reports, snapshots, window management, batch automation, CLI usage, YAML test authoring, and result-json triage
- **MCP prompts** — three server-hosted prompts (usage, testing, debugging) give the agent contextual guidance without extra configuration
- **Application branching** — application-specific automation logic (e.g. Firefox, Electron, Win32/MFC) is isolated into branches that enrich discovery, filter instances, and classify blocker surfaces without polluting the generic pipeline
- **Live observability** — the `observe` tool lets agents subscribe to continuous UI change events (structure changes, property changes, focus changes, invocations) so they can react to UI state transitions without polling
- **Multi-app orchestration** — session groups coordinate multiple connected apps with named roles, and batch steps can target per-step `sessionId` overrides for cross-application workflows within a single batch call

---

## Quick Start

### Install

Download the appropriate Drive.NET bundle for your machine, extract it, and run the installer from PowerShell:

```powershell
.\Install-DriveNet.ps1
```

The guided installer walks you through choosing an install location and optionally configuring your first workspace. By default Drive.NET installs into `%LOCALAPPDATA%\DriveNet` and adds the CLI to the user `PATH`.

After installation, **DriveNet Helper** launches automatically — use it to add workspaces, scan directories, and manage the installation going forward.

### Managing Workspaces

Open DriveNet Helper from the install directory at any time. It provides:

- **First-run guided setup** — add a single workspace or scan a directory tree to find all your repositories
- **Workspace dashboard** — see health status at a glance and fix broken configurations
- **Server management** — view server status, start/stop the server, and browse logs
- **Diagnostics** — run `doctor` checks to verify the environment
- **Updates** — check for new Drive.NET releases, download and apply them, and refresh workspaces afterward
- **Uninstall** — clean removal of Drive.NET and workspace configurations

For non-interactive installs or CI/automation, the bundled PowerShell scripts are also available:

```powershell
# Install and configure a workspace in one step (non-interactive)
.\Install-DriveNet.ps1 -WorkspaceRoot C:\Source\MyApp -NonInteractive

# Configure an additional workspace against an existing install
.\Install-DriveNet.ps1 -Mode ConfigureWorkspace -WorkspaceRoot C:\Source\AnotherApp -ExistingInstallRoot C:\Tools\DriveNet -NonInteractive

# Scan a parent directory and refresh every detected workspace
.\Update-DriveNetWorkspaces.ps1 -ScanRoot C:\Source -NonInteractive

# Update or uninstall
.\Update-DriveNet.ps1 -NonInteractive
.\Uninstall-DriveNet.ps1 -NonInteractive
```

### First Use

1. Start the target desktop application.
2. Ensure your workspace has a Drive.NET MCP entry in `.vscode/mcp.json` (Helper configures this automatically).
3. Verify that Drive.NET can see the desktop application from a terminal:

```powershell
DriveNet.Cli.exe doctor
DriveNet.Cli.exe discover
```

4. Ask your AI agent or Copilot to connect and interact with the app.

Example prompts:

- `Connect to MyApp, list the available windows, and retarget to the settings dialog if it opens`
- `Find the Submit button and click it`
- `Inspect the username field and type 'alice@example.com'`
- `Capture the current dialog after clicking Save`
- `Run an accessibility analysis report against MyApp and export as SARIF`
- `Create an analysis snapshot, fix the flagged issues, then compare the before/after snapshots`

---

## Documentation

- [Detailed Guide](docs/README.md) — installation, architecture, MCP configuration, and operational details
- [Tool Index](docs/tool-index.md) — per-tool documentation index for all MCP tools and CLI commands
- [Usage Examples](docs/usage-examples.md) — practical end-to-end workflows
- [VS Code MCP Configuration](docs/vscode-mcp-config.md) — manual MCP server setup and troubleshooting
- [YAML Test Runner Guide](docs/yaml-test-runner.md) — manifests, suites, expectations, JSON reports, and troubleshooting

## Architecture

```
┌─────────────────────┐     stdio      ┌─────────────────────────────┐
│  AI Agent / Copilot │◄──────────────►│  Drive.NET MCP Server       │
│  (MCP Client)       │                │  ┌─────────────────────────┐ │
└─────────────────────┘                │  │  MCP Layer (17 Tools,   │ │
                                       │  │  4 Resources, 3 Prompts)│ │
                                       │  ├─────────────────────────┤ │
                                       │  │  Session Manager        │ │
                                       │  │  (multi-app tracking)   │ │
                                       │  ├─────────────────────────┤ │
                                       │  │  UIA Engine             │ │
                                       │  │  (element query,        │ │
                                       │  │   interaction,          │ │
                                       │  │   event subscription)   │ │
                                       │  └─────────┬───────────────┘ │
                                       └────────────┼─────────────────┘
                                                    │ UIA / COM
                                       ┌────────────▼─────────────────┐
                                       │  Target Desktop Application  │
                                       │  (WinForms / WinUI / WPF /   │
                                       │   Electron / Win32 / etc.)   │
                                       └──────────────────────────────┘
```

## Requirements

- Windows 10 or Windows 11
- A running target application with a Microsoft UI Automation surface
