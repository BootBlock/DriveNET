# Drive.NET

Drive.NET is a Windows desktop automation toolkit built for agentic AI. It exposes .NET desktop applications built with WinForms or WinUI 3+ through Model Context Protocol (MCP), so AI agents — such as GitHub Copilot, Claude, or any MCP-compatible client — can discover processes, inspect UI trees, interact with controls, capture visual evidence, run accessibility analysis, and execute deterministic multi-step UI workflows.

While Drive.NET can be used manually through its CLI, it is ideally driven by an agentic AI that can plan, observe, and react to a live desktop application in real time. The MCP server gives the agent a structured, tool-based interface to the full Windows UI Automation surface — no screenshots-only guessing, no pixel matching, and no code injection into the target app.

---

## Features

### Process Discovery & Session Management

- Find running desktop processes with visible windows, optional window inventories, and parent/child process hierarchies
- Filter by process name, .NET-only, or application-specific branches (e.g. Firefox)
- Attach to a target application through a reusable session; retarget to secondary windows, dialogs, or flyouts as workflow demands
- Session-start warning toast with DWM acrylic frost and a slide-in countdown before automation begins (configurable, per-session overridable)
- Window handle aliases for quick retargeting between named windows
- Connect filters: `processName`, `processId`, `windowTitleRegex`, `executablePath`, `commandLineContains`
- `connectNewest` / `connectLatest` actions for multi-instance targets

### Element Querying & Inspection

- Find UI elements by automation ID, name, control type, class name, or hierarchical path selectors (e.g. `Pane[automationId=MainPanel] > Button[name=Save]`)
- Enumerate the element tree with configurable depth (0–25) and node budgets (1–5000)
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

- Execute up to 100 `query`, `interact`, and `wait_for` steps atomically in a single tool call
- Variable binding: `saveAs` saves the first matched element ID, `save` extracts result fields via JSON path, `${name}` references in later steps
- Conditional execution: `when` with variable equality, inequality, and existence checks
- Per-step retry policies: `maxAttempts`, `delayMs`, `backoffMs`
- Timing control: `delayBetweenMs` at the batch level, `delayBeforeMs` / `delayAfterMs` per step
- Per-step error handling: `continueOnError` overrides batch-level `stopOnError`
- Reserved execution variables: `lastStepSuccess`, `lastStepSkipped`, `lastStepError`, `step1Success`, etc.
- Selector explains steps for in-batch diagnostics

### Automation Analysis & Reporting

21+ checks across Automation Readiness and Accessibility Compliance:

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
- Reports are written under the workspace `.drive-net` directory
- Framework detection: WinForms, WinUI 3, Flutter, Unknown
- Score tiers: Gold, Green, Amber, Red, or Inconclusive (when UIA surface is incomplete)
- Configurable severity overrides per check

### Snapshot Comparison

- Create `.dncsnap` analysis baselines from live sessions
- Compare two snapshots to track improvement or detect regressions
- Diff output in Markdown or JSON
- Artifacts written under the workspace `.drive-net` directory

### Application Branching

- Automation branching isolates application-specific behaviour from the generic UI automation pipeline
- **Firefox branch**: auto-enriches discovery, supports newest-instance attach filtering, deprioritises onboarding windows, classifies and dismisses first-run blocker surfaces

---

## Included Components

| Component | Description |
|---|---|
| **MCP Server** | 12 tools, 3 resources, and 3 prompts over stdio transport |
| **CLI** | One-shot commands: `doctor`, `discover`, `desktop`, `windows`, `find`, `inspect`, `interact`, `tree`, `wait_for`, `batch`, `capture`, `report`, `snapshot`, `test` |
| **Companion** | WinUI 3 desktop app for interactive accessibility analysis, element exploration, code generation, and snapshot comparison |
| **Analysis Engine** | Shared analysis, report, and snapshot tooling used by MCP, CLI, and Companion |
| **Copilot Skills** | 12 workspace-scoped skill files for targeted agent guidance |
| **Installer** | Install, update, uninstall, per-workspace MCP configuration, and artifact cleanup |

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
| `batch` | Execute up to 100 query/interact/wait_for steps atomically with variable binding | No |
| `report` | Run automation analysis and write Markdown/JSON/HTML/SARIF artifacts | No |
| `snapshot` | Create or compare reusable `.dncsnap` analysis baselines | No |

### Resources

| Resource | URI | Description |
|---|---|---|
| `sessions` | `drivenet://sessions` | Enumeration of active sessions with status |
| `session-windows` | `drivenet://session/{sessionId}/windows` | Top-level windows for a connected session |
| `session-element-tree` | `drivenet://session/{sessionId}/tree` | Live element tree for inspection-first diagnostics |

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
| `desktop` | Query monitors and foreground window |
| `windows` | List/manage windows: list, blockers, resize, move, minimize, maximize, restore, close, bringToFront |
| `find` | Find matching UI elements with flat or hierarchical selectors |
| `inspect` | Verbose single-element detail: patterns, value, toggle state |
| `tree` | Render UI Automation hierarchy with depth/node budgets and truncation hints |
| `interact` | Perform actions: click, doubleClick, rightClick, type, clear, sendKeys, select, toggle, expand, collapse, scrollIntoView, dragTo, mouseMove, setFocus, highlight, clipboard |
| `wait_for` | Wait for UI/window conditions with configurable timeout |
| `batch` | Execute multi-step JSON sequences atomically with variable binding |
| `capture` | Screenshot windows or elements to PNG |
| `report` | Run analysis pipeline and write report artifacts |
| `snapshot` | Create or compare analysis baselines |
| `test` | Run YAML test suites or manifests with structured result output |

### Quick Examples

```powershell
DriveNet.Cli.exe doctor
DriveNet.Cli.exe discover --dotnet-only
DriveNet.Cli.exe desktop --action monitors
DriveNet.Cli.exe find --process-name MyApp --automation-id SubmitButton
DriveNet.Cli.exe find --process-name MyApp --path "Pane[automationId=MainPanel] > Button[name=Save]" --explain
DriveNet.Cli.exe interact --process-name MyApp --action click --automation-id SubmitButton
DriveNet.Cli.exe wait_for --process-name MyApp --condition elementExists --automation-id SuccessLabel
DriveNet.Cli.exe capture --process-name MyApp --automation-id SubmitButton
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
- **Step tools**: `discover`, `query`, `interact`, `wait_for`, `window`, `capture`
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

A standalone WinUI 3 desktop application for interactive accessibility and automation-readiness analysis.

### Analysis

- **21+ analysis checks** across Automation Readiness (DNC001–DNC009, DNC016–DNC018) and Accessibility Compliance (DNC010–DNC015, DNC019–DNC021)
- **Score tiers**: Gold, Green, Amber, Red, and Inconclusive (when the UIA surface is incomplete, such as hosted shell chrome or custom-rendered content)
- **Framework detection**: WinForms, WinUI 3, Flutter, Unknown — with framework-specific remediation guidance
- **Severity filtering**: view findings by severity level
- **Watch mode**: monitor UI changes in real-time during an active session
- **Configurable severity overrides** per check

### Export

- **Markdown** — human-readable findings with recommendations
- **JSON** — machine-parseable findings for tooling
- **HTML** — rich formatted report
- **SARIF v2.1.0** — for code-scanning and CI workflows
- **AI Prompt** — generated remediation guidance with framework-specific recommendations

### Interactive Tools

- **Tree Explorer** — visualise UI Automation element hierarchy with search, property viewing, and artifact annotation
- **Element Picker** — crosshair selection tool with hover highlighting, element flash, and timeout-based auto-dismissal
- **Interaction Tester** — test element interactions with action logging
- **Code Generation** — generate selector code from picked elements and supported patterns
- **Pattern Coverage Map** — display UIA pattern availability matrix across the element tree
- **Snapshot Comparison** — create baselines and diff them to track improvement over time

### UI & Navigation

- Full keyboard navigation with configurable shortcuts
- Context menus with copy-to-clipboard actions
- Process search and filtering with process icon display and caching
- Status bar with framework badge and running state indicator
- DWM acrylic frost styling

### Special Modes

- **Self-analysis mode** (`--connect-self-analysis`): analyse the Companion's own UI for deterministic regression testing

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
- **Application branching** — application-specific automation logic (e.g. Firefox) is isolated into branches that enrich discovery, filter instances, and classify blocker surfaces without polluting the generic pipeline

---

## Quick Start

### Install

Download the appropriate Drive.NET bundle for your machine, extract it, and run the installer from PowerShell:

```powershell
.\Install-DriveNet.ps1 -WorkspaceRoot C:\Source\MyApp
```

This installs the server and CLI, writes `.vscode/mcp.json` for the target workspace, and installs the bundled Copilot skills.

For a guided interactive setup:

```powershell
.\Install-DriveNet.ps1
```

By default, Drive.NET installs into `%LOCALAPPDATA%\DriveNet` and adds the CLI to the user `PATH`.

For a custom install location:

```powershell
.\Install-DriveNet.ps1 -InstallRoot C:\Tools\DriveNet -NonInteractive
```

To configure additional workspaces against an existing install:

```powershell
.\Install-DriveNet.ps1 -Mode ConfigureWorkspace -WorkspaceRoot C:\Source\AnotherApp -ExistingInstallRoot C:\Tools\DriveNet -NonInteractive
```

To update or uninstall:

```powershell
.\Update-DriveNet.ps1 -NonInteractive
.\Uninstall-DriveNet.ps1 -NonInteractive
```

### First Use

1. Start the target WinForms or WinUI application.
2. Ensure your workspace has a Drive.NET MCP entry in `.vscode/mcp.json`.
3. Ask your AI agent or Copilot to connect and interact with the app.

Example prompts:

- `Connect to MyWinFormsApp, list the available windows, and retarget to the settings dialog if it opens`
- `Find the Submit button and click it`
- `Inspect the username field and type 'alice@example.com'`
- `Capture the current dialogue after clicking Save`
- `Run an accessibility analysis report against MyApp and export as SARIF`
- `Create an analysis snapshot, fix the flagged issues, then compare the before/after snapshots`

---

## Documentation

- [Detailed Guide](docs/README.md) — installation, architecture, MCP configuration, and operational details
- [Tool Reference](docs/tool-reference.md) — all MCP tools and CLI commands with parameters, responses, examples, and tips
- [Usage Examples](docs/usage-examples.md) — practical end-to-end workflows
- [VS Code MCP Configuration](docs/vscode-mcp-config.md) — manual MCP server setup and troubleshooting
- [YAML Test Runner Guide](docs/yaml-test-runner.md) — manifests, suites, expectations, JSON reports, and troubleshooting
- [VS Code MCP Configuration](docs/vscode-mcp-config.md): workspace configuration examples and troubleshooting
- [Changelog](CHANGELOG.md): notable changes across releases

## Architecture

```
┌─────────────────────┐     stdio      ┌──────────────────────────────┐
│  AI Agent / Copilot │◄──────────────►│  Drive.NET MCP Server        │
│  (MCP Client)       │                │  ┌─────────────────────────┐ │
└─────────────────────┘                │  │  MCP Layer (11 Tools,   │ │
                                       │  │  3 Resources, 3 Prompts)│ │
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
                                       │  Target .NET Application(s)  │
                                       │  (WinForms / WinUI 3+)       │
                                       └──────────────────────────────┘
```

## Runtime Defaults

- The MCP server writes daily-rotated logs under `%LOCALAPPDATA%\DriveNet\logs\drivenet.log` by default instead of beside the installed binaries or published bundle.
- Override the log file path, retention, or stderr threshold through `src/DriveNet.Server/appsettings.json` or environment variables such as `DriveNet__Logging__LogFilePath` and `DriveNet__Logging__StandardErrorFromLevel`.
- CLI and MCP artifact output remains workspace-scoped under `.drive-net` for screenshots, reports, and snapshots.

## Security

- **No code injection** — Drive.NET operates externally via Microsoft UI Automation; it does not inject code into target applications
- **Session isolation** — each session is scoped to a single process; element IDs are session-scoped and cannot cross sessions
- **Input validation** — all tool inputs are validated at the boundary; path parameters are checked for traversal attacks; numeric parameters are clamped to safe ranges
- **Clipboard access** — runs on a dedicated STA thread, only when explicitly requested via `interact clipboard`
- **Graceful shutdown** — on SIGTERM/SIGINT, all sessions are disconnected, UIA event handlers are unsubscribed, and logs are flushed

## Dependency Note

- The test projects currently keep `FluentAssertions` 8.x for readability. That package now carries the Xceed community/commercial license, so contributors and downstream commercial users should review that license posture before redistributing or extending the test stack.

## Requirements

- Windows 10 or Windows 11
- A running target application built with WinForms or WinUI 3+
