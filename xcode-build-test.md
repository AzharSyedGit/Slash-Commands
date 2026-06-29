---
name: xcode-mcp
description: >
  Build, test, simulator, project-edit, and verification workflows for Xcode
  driven exclusively through MCP servers (xcode-mcp / xcode-native-mcp) instead
  of raw shell commands. Use when the user asks to build, run, test, clean,
  launch on a simulator, capture logs/screenshots, measure coverage, edit Xcode
  project files, run Swift snippets, or render SwiftUI previews. Also use when a
  build/test fails and needs diagnosis, or when MCP servers need connecting or
  troubleshooting. Scoped to Xcode 26 by default; Xcode 27 beta tools gated off
  until the active version is Xcode 27.
allowed-tools: Read Grep Glob
metadata:
  version: "1.0"
  default-xcode-version: "26"
---

# Xcode MCP Skill

Prefer MCP tools over raw shell for ALL build, test, simulator, and project-edit
operations. MCP returns structured JSON: fewer tokens, better error diagnosis
than unstructured Bash output. Staying on MCP keeps workflows consistent across
build, test, project edits, and verification.

Two MCP servers back this skill:

| Server | Scope |
| --- | --- |
| `xcode-mcp` (xcodebuildmcp) | build, test, simulator, logs, coverage |
| `xcode-native-mcp` | live Xcode project edits, build log, tests, previews, snippets |

---

## When to Use

- Building for a simulator (compile-only or build + install + launch).
- Running tests (full test plan or a targeted subset) on a simulator.
- Cleaning build products.
- Enumerating, booting, or opening simulators.
- Managing app lifecycle on a simulator (install, launch, stop, bundle id).
- Capturing logs, screenshots, video, or a UI view hierarchy.
- Measuring per-target or per-file code coverage from an xcresult bundle.
- Discovering projects/schemes/build settings or setting session defaults.
- Editing files that must be part of an Xcode target.
- Verifying Swift snippets or rendering SwiftUI previews headlessly.
- Diagnosing a failed build or test from structured output.
- Connecting or troubleshooting the MCP servers themselves.

## When Not to Use

- Device builds or device tests: not exposed by MCP in this workspace. Ask the
  user before running anything on a physical device.
- Editing `.pbxproj`, anything inside `.xcodeproj/`, dependency manifests,
  deployment targets, or entitlements: out of scope; do not modify these.
- Pure source edits that do not need Xcode target membership: use ordinary file
  tools.
- Debugger-driven runtime inspection: no debugger MCP tool under Xcode 26 (use
  `launch_app_sim` logs and `snapshot_ui`). Under Xcode 27, use
  `InvokeDebuggerCommand` / `GetConsoleOutput`.
- Apple API documentation lookups: no Apple-docs MCP tool under Xcode 26 (fall
  back to web search against `developer.apple.com`). Under Xcode 27, use
  `DocumentationSearch`.

---

## Prerequisites

Verify both MCP servers are connected before any work:

- `xcode-native-mcp`
- `xcode-mcp`

Do not begin any build, test, simulator, project-edit, or verification work
until both are confirmed connected. See Troubleshooting below for fixes.

Call `session_show_defaults` before the first build/run/test in a session.

### Server Availability — Blocking Precondition

Do not proceed until both servers are confirmed accessible. This is a hard gate:

- Before any build, test, simulator, project-edit, or verification work, confirm
  both servers are connected and their tools are reachable.
- If either server is unavailable, stop and tell the user. Do NOT fall back to
  raw shell commands (`xcodebuild`, `xcrun simctl`, `swift`, etc.) to work
  around an inaccessible server.
- Resume only after both servers are confirmed accessible.

---

## Tool Reference (Xcode 26)

These tools are the default tool set, active when the active Xcode version is 26.

### Build

| Tool | Description |
| --- | --- |
| `build_sim` (xcode-mcp) | Simulator build, compile-only. |
| `build_run_sim` (xcode-mcp) | Build + install + launch in one step. |
| `BuildProject` (xcode-native-mcp) | Build via the active Xcode workspace. |
| `GetBuildLog` (xcode-native-mcp) | Inspect the most recent build log. |
| `clean` (xcode-mcp) | Clean build products. |

Do NOT invoke `xcodebuild` via Bash. Device builds are not exposed by MCP; ask
the user before running a device build.

### Test

| Tool | Description |
| --- | --- |
| `test_sim` (xcode-mcp) | Run simulator tests. |
| `RunAllTests` (xcode-native-mcp) | Run the active scheme's test plan. |
| `RunSomeTests` (xcode-native-mcp) | Run a targeted subset of tests. |
| `GetTestList` (xcode-native-mcp) | Discover test identifiers before `RunSomeTests`. |

Do NOT invoke `xcodebuild test` via Bash. Device tests are not exposed by MCP;
ask the user before running on device.

### Simulators

| Tool | Description |
| --- | --- |
| `list_sims` | Enumerate simulators. |
| `boot_sim` | Boot a simulator (not required before `build_run_sim`). |
| `open_sim` | Open Simulator.app (not required before `build_run_sim`). |

Do NOT invoke `xcrun simctl` via Bash.

### App Lifecycle (simulator)

| Tool | Description |
| --- | --- |
| `get_sim_app_path` | Resolve the built app bundle path. |
| `install_app_sim` | Install an app on the simulator. |
| `launch_app_sim` | Launch an installed app; runtime logs are captured automatically and the log file path is returned. |
| `stop_app_sim` | Stop a running app. |
| `get_app_bundle_id` | Read the bundle id from an app bundle. |

### Logs, Screenshots, UI

| Tool | Description |
| --- | --- |
| `record_sim_video` | Video capture (MP4). |
| `screenshot` | Simulator screenshot. |
| `snapshot_ui` | Semantic UI snapshot / view hierarchy with elementRef targets. |

For runtime logs, use `launch_app_sim` (logs captured automatically) — there is
no standalone log-capture tool.

### Coverage

| Tool | Description |
| --- | --- |
| `get_coverage_report` | Per-target coverage from an xcresult bundle. |
| `get_file_coverage` | Function-level coverage of a specific file. |

### Project Discovery & Defaults

| Tool | Description |
| --- | --- |
| `discover_projs` | Find projects/workspaces. |
| `list_schemes` | List schemes. |
| `show_build_settings` | Show resolved build settings. |
| `session_show_defaults` | Show current session defaults. |
| `session_set_defaults` | Set session defaults. |
| `session_clear_defaults` | Clear session defaults. |
| `session_use_defaults_profile` | Switch the active defaults profile. |

### Xcode Project Edits (xcode-native-mcp)

Operate on the live Xcode project organization, not raw filesystem paths. Prefer
these when working on files that must be part of the Xcode target.

| Tool | Description |
| --- | --- |
| `XcodeRead` | Read a file in the Xcode project. |
| `XcodeLS` | List entries in the Xcode project. |
| `XcodeGlob` | Glob-match files in the Xcode project. |
| `XcodeGrep` | Search file contents in the Xcode project. |
| `XcodeWrite` | Create or overwrite a project file. |
| `XcodeUpdate` | Edit an existing project file. |
| `XcodeMV` | Move or rename a project file. |
| `XcodeRM` | Remove a project file. |
| `XcodeMakeDir` | Create a directory in the project. |
| `XcodeRefreshCodeIssuesInFile` | Refresh diagnostics for a file. |
| `XcodeListNavigatorIssues` | List navigator issues. |
| `XcodeListWindows` | List windows (for the `tabIdentifier` parameter). |

Editing guardrails still apply: do NOT modify `.pbxproj`, anything inside
`.xcodeproj/`, dependency manifests, deployment targets, or entitlements.

### Swift Verification

| Tool | Description |
| --- | --- |
| `RunCodeSnippet` (xcode-native-mcp) | Run and verify Swift snippets in a source file's context. |

Do NOT invoke `swift` via Bash.

### SwiftUI Previews

| Tool | Description |
| --- | --- |
| `RenderPreview` (xcode-native-mcp) | Headless SwiftUI verification. |

### Apple Documentation

| Tool | Description |
| --- | --- |
| (none) | No Apple documentation MCP tool is available. Fall back to web search against `developer.apple.com` for Apple API lookups. |

### Debugging

| Tool | Description |
| --- | --- |
| (none) | No debugger MCP tools are available. For runtime inspection, use `launch_app_sim` (auto log capture) and `snapshot_ui`. |

---

## Tool Reference (Xcode 27)

**Version gate:** Tools in this section carry the availability tag
`available for Xcode 27`. They MUST NOT be used or treated as active when the
active Xcode version is 26. Activate them only when the active version is
Xcode 27. When Xcode 26 is active, ignore this section entirely and use the
Xcode 26 tool set above.

All tools below are on `xcode-native-mcp` and surfaced when Xcode 27 became the
active toolchain (verified live on build `27A5194q`: 47 `xcode-native-mcp`
tools total — the 19 Xcode 26 tools above plus the 28 below). `xcode-mcp` is
unchanged at 24 tools. Add future Xcode 27 tools as new rows in the matching
category table, preserving the `available for Xcode 27` tag.

### Run & Debug (Xcode 27)

| Tool | Description | Availability |
| --- | --- | --- |
| `RunProject` | Build + run the active scheme (Cmd+R); optionally attach the debugger. | available for Xcode 27 |
| `StopProject` | Stop the currently running app (Cmd+.). | available for Xcode 27 |
| `InvokeDebuggerCommand` | Send an lldb command to Xcode's active debug session. | available for Xcode 27 |
| `GetConsoleOutput` | Retrieve stdout/stderr/OSLog from a launch session, with regex/severity filters. | available for Xcode 27 |

### Schemes & Run Destinations (Xcode 27)

| Tool | Description | Availability |
| --- | --- | --- |
| `XcodeListSchemes` | List schemes and identify the active one. | available for Xcode 27 |
| `XcodeSwitchScheme` | Change the active scheme. | available for Xcode 27 |
| `XcodeListRunDestinations` | List run destinations for the active scheme. | available for Xcode 27 |
| `XcodeSwitchRunDestination` | Change the active run destination. | available for Xcode 27 |

### Target & Build Configuration (Xcode 27)

| Tool | Description | Availability |
| --- | --- | --- |
| `GetTargetBuildSettings` | Read build settings for a target. | available for Xcode 27 |
| `UpdateTargetBuildSetting` | Update/append/delete a target build setting. | available for Xcode 27 |
| `GetFileCompilerFlags` | Read per-file compiler flags. | available for Xcode 27 |
| `UpdateFileCompilerFlags` | Update/append/delete per-file compiler flags. | available for Xcode 27 |
| `AddEntitlement` | Add a code-signing entitlement to a target. | available for Xcode 27 |
| `AddInfoPlist` | Add/update an Info.plist key for a target. | available for Xcode 27 |

### Project Edits (Xcode 27)

| Tool | Description | Availability |
| --- | --- | --- |
| `XcodeGetCurrentFile` | Get the active editor file, content, and selection. | available for Xcode 27 |

### Localization & String Catalogs (Xcode 27)

| Tool | Description | Availability |
| --- | --- | --- |
| `LocalizationPlanner` | Prepare the project to add translations for a locale. | available for Xcode 27 |
| `StringCatalogRead` | Read string keys grouped by translation state. | available for Xcode 27 |
| `StringCatalogContext` | Get source value and context for a string key. | available for Xcode 27 |
| `StringCatalogEdit` | Insert a translation for a locale. | available for Xcode 27 |

### Device Interaction (Xcode 27)

| Tool | Description | Availability |
| --- | --- | --- |
| `DeviceInteractionStartSession` | Prepare/boot a device or simulator for interaction. | available for Xcode 27 |
| `DeviceInteractionEndSession` | Close a device interaction session. | available for Xcode 27 |
| `DeviceInteractionInstallAndRun` | Build, install, and start the app on the targeted device. | available for Xcode 27 |
| `DeviceInteractionSynthesize` | Synthesize tap/swipe/type/press events and capture state. | available for Xcode 27 |

### Crash & Field Diagnostics (Xcode 27)

| Tool | Description | Availability |
| --- | --- | --- |
| `GetTopCrashIssues` | Top crash signatures by device count. | available for Xcode 27 |
| `GetCrashIssueLogs` | Detailed crash logs and triage guidance for a signature. | available for Xcode 27 |
| `GetTopFieldPerformanceIssues` | Top field performance issues (launches, hangs, disk writes, energy). | available for Xcode 27 |
| `GetFieldPerformanceIssueLogs` | Detailed logs/triage for a field performance signature. | available for Xcode 27 |

### Apple Documentation (Xcode 27)

| Tool | Description | Availability |
| --- | --- | --- |
| `DocumentationSearch` | Search Apple Developer Documentation semantically. | available for Xcode 27 |

---

## Troubleshooting

### `xcode-native-mcp` not connected

Direct the user to enable Xcode's built-in MCP server:

1. Open Xcode.
2. Navigate to Settings (⌘,) > Intelligence > Model Context Protocol.
3. Ensure the MCP toggle (labeled **Xcode Tools**) is switched **ON**. This tells
   Xcode to accept incoming MCP connections from external agents.

Additional notes:

- Apple's built-in Xcode MCP server shipped in **Xcode 26.3**. If the
  Intelligence > Model Context Protocol section is missing, the user is likely on
  an older Xcode; have them update to 26.3 or later.
- The toggle exposes Xcode's tools (project hierarchy queries, builds, tests,
  previews, snippets) to external agents over MCP.
- After enabling the toggle, the project/workspace must be open in Xcode for the
  server to expose live tools. Reconnect the MCP server (or restart the client)
  if it was started before the toggle was enabled.

### "Connected" but zero tools (macOS Automation permission)

If `xcode-native-mcp` shows **Connected** but exposes **zero tools**, usually
with a repeating macOS pop-up asking to **Allow access to Xcode**, this is a
macOS TCC (Automation) permission problem — the client can connect but can't
control Xcode, so it can't enumerate any tools. To fix:

1. When the macOS pop-up appears, click **Allow** (it does not always persist, so
   the steps below make it stick).
2. Open **System Settings > Privacy & Security > Automation**.
3. Find the MCP client app (e.g., Kiro / Cursor / the host editor) and ensure its
   toggle for **Xcode** is enabled.
4. If it keeps reprompting or still shows zero tools, fully quit and relaunch both
   the client and Xcode, then reconnect the MCP server.
5. As a last resort, reset the Automation permission, then re-grant on next
   prompt:
   - Reset for the specific client bundle id: `tccutil reset AppleEvents <client-bundle-id>`
   - Or reset all AppleEvents permissions: `tccutil reset AppleEvents`

Notes: this affects multiple MCP clients. The prompt can reappear on restart or
multiple times per session because separate helper/plugin processes each trigger
their own permission request, and CLI clients without a bundle identifier can
hang on the permission step.
