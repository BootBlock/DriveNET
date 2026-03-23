# Drive.NET

Drive.NET is a Windows desktop automation toolkit built for agentic AI. It exposes .NET desktop applications built with WinForms or WinUI 3+ through Model Context Protocol (MCP), so AI agents — such as GitHub Copilot, Claude, or any MCP-compatible client — can discover processes, inspect UI trees, interact with controls, capture visual evidence, run accessibility analysis, and execute deterministic multi-step UI workflows.

> While the Drive.NET Companion application can detect and partially analyse Flutter applications, they're not yet supported for automation.

While Drive.NET can be used manually through its CLI, it is ideally driven by an agentic AI that can plan, observe, and react to a live desktop application in real time. The MCP server gives the agent a structured, tool-based interface to the full Windows UI Automation surface — no screenshots-only guessing, no pixel matching, and no code injection into the target app.

## What It Does

- **Process discovery** — find running desktop processes with visible windows, optional window inventories, and parent/child process hierarchies
- **Session management** — attach to a target application through a reusable session; retarget to secondary windows, dialogs, or flyouts as workflow demands
- **Element querying** — find UI elements by automation ID, name, control type, class name, or hierarchical path selectors; enumerate the element tree with configurable depth and node budgets; read properties, children, parents, and grid/table data
- **Element inspection** — get exhaustive information about a single element including all UIA properties, supported patterns, and available actions
- **UI interaction** — click, double-click, right-click, type, clear, sendKeys, select, toggle, expand, collapse, scrollIntoView, drag, setFocus, highlight, and clipboard read/write
- **Condition waiting** — wait for element existence/removal, property changes, text equality, element enabled/visible state, window open/close, and structure changes — with polling or event-driven detection
- **Window management** — list windows, detect blocking modal dialogs with suggested resolution actions, resize, move, minimize, maximize, restore, close, and bring to front
- **Screenshot capture** — capture entire windows or individual elements as inline base64 MCP images or saved PNG files
- **Batch automation** — execute up to 100 query/interact/wait_for steps atomically with variable binding (`saveAs`/`save`/`${name}`), conditional execution (`when`), per-step retries, timing control, and per-step error handling
- **Selector diagnostics** — explain how a selector resolves before acting, so the agent can verify targeting without side effects
- **Automation analysis & reporting** — run 21+ checks across automation readiness and WCAG accessibility compliance; export as Markdown, JSON, HTML, or SARIF
- **Snapshot comparison** — create `.dncsnap` baselines and diff them to track improvement or detect regressions
- **YAML test runner** — author deterministic multi-step UI test suites in YAML with manifests, expectations, saved variables, and machine-readable JSON result output for agentic triage

## Designed for Agentic AI

Drive.NET is purpose-built so an AI agent can autonomously operate a desktop application:

- **Structured tool interface** — every operation is a typed MCP tool call with validated inputs and structured JSON responses, not free-form screen scraping
- **In-band tree truncation** — large UI trees are bounded by a configurable node budget and return continuation hints so the agent can drill into subtrees without oversized payloads
- **Selector explain** — the agent can verify a selector resolves correctly before committing to a click or wait, reducing wasted actions
- **Blocker detection** — `window blockers` detects modal dialogs and returns ranked suggested actions (with confidence levels) so the agent can unblock itself
- **Variable binding in batch** — multi-step workflows can save element IDs and result fields into named variables, enabling data-dependent automation in a single tool call
- **Built-in Copilot skills** — 12 workspace-scoped skills provide domain-specific guidance for discovery, querying, interaction, waiting, capture, reports, snapshots, window management, batch automation, CLI usage, YAML test authoring, and result-json triage
- **MCP prompts** — three server-hosted prompts (usage, testing, debugging) give the agent contextual guidance without extra configuration

## Included Components

| Component | Description |
|---|---|
| **MCP Server** | Exposes 11 tools, 3 resources, and 3 prompts over stdio transport |
| **CLI** | Fast one-shot commands for doctor, discover, windows, find, inspect, interact, tree, wait_for, batch, capture, report, snapshot, and YAML test workflows |
| **Companion** | WinUI 3 desktop app for interactive accessibility analysis with 21+ checks, 5 export formats, element highlighting, crosshair picker, code generation, and snapshot comparison |
| **Analysis Engine** | Shared report and snapshot tooling available from MCP, CLI, and Companion surfaces |
| **Copilot Skills** | 12 workspace-scoped skill files for targeted agent guidance |
| **Installer Scripts** | Install, update, uninstall, workspace MCP configuration, and artifact cleanup helpers |

## MCP Tools

| Tool | Description | Read-Only |
|---|---|---|
| `discover` | List running processes, optional window inventories and process hierarchies | Yes |
| `session` | Connect to, retarget, or disconnect from a target application | No |
| `query` | Find/resolve elements, enumerate trees, read properties, grid data, explain selectors | Yes |
| `inspect` | Exhaustive element info: properties, supported patterns, available actions | Yes |
| `interact` | Click, type, select, toggle, expand, collapse, drag, sendKeys, clipboard, highlight | No |
| `wait_for` | Wait for UI conditions with polling or event-driven detection; explain selectors | Yes |
| `window` | List windows, detect blockers, resize, move, minimize, maximize, restore, close | No |
| `capture` | Screenshot windows or elements as base64 inline images or PNG files | No |
| `batch` | Execute up to 100 query/interact/wait_for steps atomically with variable binding | No |
| `report` | Run automation analysis and write Markdown/JSON/HTML/SARIF artifacts | No |
| `snapshot` | Create or compare reusable `.dncsnap` analysis baselines | No |

## Quick Start

### Use The Bundled Release

1. Download the appropriate Drive.NET bundle for your machine.
2. Extract it.
3. Run the installer once from PowerShell:

```powershell
.\Install-DriveNet.ps1 -WorkspaceRoot C:\Source\MyApp
```

That installs the server and CLI, writes `.vscode/mcp.json` for the target workspace, and installs the bundled Copilot skills.

### Install Once, Configure Workspaces Later

If you want Drive.NET installed once in a stable location and then reused across multiple workspaces, the recommended end-user flow is to run the installer interactively and choose the install location you want:

```powershell
.\Install-DriveNet.ps1
```

From the guided installer, choose the install or update option and skip workspace configuration until you are ready to wire up a specific workspace.

By default, Drive.NET installs into `%LOCALAPPDATA%\DriveNet` and adds that location to the user `PATH`.

If you already know the exact install location you want and are scripting the install, you can still run it directly:

```powershell
.\Install-DriveNet.ps1 -InstallRoot C:\Tools\DriveNet -NonInteractive
```

Then configure each workspace separately against that installed location:

```powershell
.\Install-DriveNet.ps1 -Mode ConfigureWorkspace -WorkspaceRoot C:\Source\MyApp -ExistingInstallRoot C:\Tools\DriveNet -NonInteractive
```

Drive.NET installs the server and CLI once, but MCP configuration is workspace-scoped. The installer updates `.vscode/mcp.json` for each workspace you choose to configure.

### If The Installer Hits A Problem

The guided and scripted installers now end with a structured failure summary and recovery steps instead of only a raw PowerShell error. The most common fixes are:

1. Re-run the installer from a fully extracted Drive.NET bundle that still contains `DriveNet.Server.exe`, `DriveNet.Cli.exe`, and the `copilot` folder.
2. Make sure the workspace path already exists, or use the guided installer and let it create the workspace folder for you.
3. For `ConfigureWorkspace`, point `-ExistingInstallRoot` at an actual installed Drive.NET directory rather than the extracted release folder.
4. If `.vscode\mcp.json` or workspace skill files appear locked, close VS Code and rerun the installer.

### First Use

1. Start the target WinForms or WinUI application (or ask your agent to launch it).
2. Ensure your workspace has a Drive.NET MCP entry.
3. Ask your AI agent or Copilot to connect and interact with the app.

Example prompts:

- `Connect to MyWinFormsApp, list the available windows, and retarget to the settings dialog if it opens`
- `Find the Submit button and click it`
- `Inspect the username field and type 'alice@example.com'`
- `Capture the current dialogue after clicking Save`
- `Run an accessibility analysis report against MyApp and export as SARIF`
- `Create an analysis snapshot, fix the flagged issues, then compare the before/after snapshots`

## CLI Quick Examples

```powershell
DriveNet.Cli.exe doctor
DriveNet.Cli.exe discover --dotnet-only
DriveNet.Cli.exe find --process-name MyApp --automation-id SubmitButton
DriveNet.Cli.exe find --process-name MyApp --path "Pane[automationId=MainPanel] > Button[name=Save]" --explain
DriveNet.Cli.exe interact --process-name MyApp --action click --automation-id SubmitButton
DriveNet.Cli.exe wait_for --process-name MyApp --condition elementExists --automation-id SuccessLabel
DriveNet.Cli.exe capture --process-name MyApp --automation-id SubmitButton
DriveNet.Cli.exe report --process-name MyApp --format markdown
DriveNet.Cli.exe snapshot create --process-name MyApp --output snapshots/latest.dncsnap
DriveNet.Cli.exe test --manifest tests/definitions/manifest.yaml --result-json results.json
```

The CLI defaults to human-readable output and supports `--json` on each command for scripting and agentic consumption.

## YAML Test Runner

Drive.NET ships a YAML test runner for deterministic, repeatable UI automation workflows:

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

Run with `--result-json` for machine-readable per-step results that an agent can triage automatically.

## DriveNet Companion

A standalone WinUI 3 desktop application for interactive accessibility and automation-readiness analysis:

- **21+ analysis checks** across Automation Readiness (DNC001–DNC009, DNC016–DNC018) and Accessibility Compliance (DNC010–DNC015, DNC019–DNC021)
- **5 export formats**: Markdown, AI prompt, JSON, SARIF v2.1.0, HTML
- **Snapshot comparison**: create baselines and diff them to track improvement over time
- **Interactive tools**: element interaction tester, code generation from selected patterns, pattern coverage map
- **Visual aids**: element highlighting overlays, crosshair picker for targeting
- **Full keyboard navigation** with configurable shortcuts
- **Self-analysis mode**: can analyse its own UI for deterministic regression testing (`--connect-self-analysis`)

## Documentation

- [Detailed Guide](docs/README.md): installation, architecture, testing, CLI, MCP configuration, and operational details
- [Tool Reference](docs/tool-reference.md): all MCP tools, parameters, responses, examples, and tips
- [Usage Examples](docs/usage-examples.md): practical end-to-end examples and workflows
- [YAML Test Runner Guide](docs/yaml-test-runner.md): manifests, suites, expectations, JSON reports, Companion examples, and troubleshooting
- [VS Code MCP Configuration](docs/vscode-mcp-config.md): workspace configuration examples and troubleshooting
- [Changelog](CHANGELOG.md): notable changes across releases

## Architecture

```
┌─────────────────────┐     stdio      ┌───────────────────────────────┐
│  AI Agent / Copilot │◄──────────────►│  Drive.NET MCP Server         │
│  (MCP Client)       │                │  ┌─────────────────────────┐  │
└─────────────────────┘                │  │  MCP Layer (11 Tools,   │  │
                                       │  │  3 Resources, 3 Prompts)│  │
                                       │  ├─────────────────────────┤  │
                                       │  │  Session Manager        │  │
                                       │  │  (multi-app tracking)   │  │
                                       │  ├─────────────────────────┤  │
                                       │  │  UIA Engine             │  │
                                       │  │  (element query,        │  │
                                       │  │   interaction,          │  │
                                       │  │   event subscription)   │  │
                                       │  └─────────┬───────────────┘  │
                                       └────────────┼──────────────────┘
                                                    │ UIA / COM
                                       ┌────────────▼─────────────────┐
                                       │  Target .NET Application(s)  │
                                       │  (WinForms / WinUI 3+)       │
                                       └──────────────────────────────┘
```

## Runtime Defaults

- The MCP server writes daily-rotated logs under `%LOCALAPPDATA%\DriveNet\logs\drivenet.log` by default instead of beside the installed binaries or published bundle.
- Override the log file path, retention, or stderr threshold through `src/DriveNet.Server/appsettings.json` or environment variables such as `DriveNet__Logging__LogFilePath` and `DriveNet__Logging__StandardErrorFromLevel`.
- CLI and MCP artefact output remains workspace-scoped under `.drive-net` for screenshots, reports, and snapshots.

## Security

- **No code injection** — Drive.NET operates externally via Microsoft UI Automation; it does not inject code into target applications
- **Session isolation** — each session is scoped to a single process; element IDs are session-scoped and cannot cross sessions
- **Input validation** — all tool inputs are validated at the boundary; path parameters are checked for traversal attacks; numeric parameters are clamped to safe ranges
- **Clipboard access** — runs on a dedicated STA thread, only when explicitly requested via `interact clipboard`
- **Graceful shutdown** — on SIGTERM/SIGINT, all sessions are disconnected, UIA event handlers are unsubscribed, and logs are flushed

## Requirements

- Windows 10 or Windows 11
- A running target application built with WinForms or WinUI 3+
