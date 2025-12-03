# BBOT TUI Viewer

A self-contained terminal UI for browsing and analyzing [BBOT](https://www.blacklanternsecurity.com/bbot/) scan results.

![BBOT TUI Demo](docs/demo.gif)

## Features

- 🚀 **Zero Setup** - Single self-installing file, no manual dependencies
- 🔴 **Live Refresh** - Auto-updates while scans are running with accurate status detection
- 🎯 **Smart Status Detection** - Accurately identifies RUNNING, FINISHED, and INTERRUPTED scans
- 📋 **Scan Browser** - Navigate multiple scans with separate vulnerability/finding counts and status indicators
- 📦 **Archive Management** - Compress old scans to save space, restore when needed
- 📝 **Work Tracking** - Annotate vulnerabilities and findings with status, priority, and notes
- 🔍 **Separate Views** - Dedicated tabs for vulnerabilities (sorted by severity) and findings
- 🌳 **Discovery Tree** - Hierarchical view showing parent-child event relationships
- 📊 **Rich Statistics** - Beautiful tables with event distribution, scope analysis, and workflow metrics
- 🔎 **Event Explorer** - Filter, search, and inspect all scan events
- ⚙️ **Config Viewer** - View preset.yml configuration

## Quick Start

```bash
# Copy to server and run (auto-installs on first run)
./bbot-ui

# Or specify custom path
./bbot-ui /path/to/scans
```

First run creates `.bbot_ui_venv/` and installs dependencies. Subsequent runs launch instantly.

## Usage

```bash
./bbot-ui                          # Default: ~/.bbot/scans
./bbot-ui /path/to/scans          # Browse all scans in directory
./bbot-ui ~/.bbot/scans/scan-name # View specific scan
```

### Command-Line Options

```bash
./bbot-ui --help                               # Show all options
./bbot-ui --scan-interval 5                    # Refresh scan view every 5 seconds
./bbot-ui --list-interval 10                   # Refresh scan list every 10 seconds
```

**Available options:**
- `--scan-interval SECONDS` - Refresh interval for scan detail view (default: 2.0)
- `--list-interval SECONDS` - Refresh interval for scan list view (default: 3.0)

Settings are automatically saved to `~/.bbot_ui_config.json` and used as defaults for future sessions.

## Interface

### Scan List
- Browse all scans in a table with columns: Scan Name, Status, Events, Vulns, Findings, Last Modified
- Header shows total scans, vulnerability/finding counts, and running scan count
- **Status column** shows real-time scan state:
  - **● RUNNING** (green) - Scan actively running with bbot process detected
  - **⚠ INTERRUPTED** (yellow) - Scan was stopped/interrupted (no active process)
  - **✓ FINISHED** (blue) - Scan completed successfully
  - **○ CHECKING...** (dim) - Status being verified (appears briefly at startup)
- Vulns and Findings columns show **⚠** indicator for scans with vulnerabilities/findings
- Auto-refreshes every 3 seconds to show new scans and status changes
- Status verified within 0.5 seconds of startup (no false "RUNNING" status)
- `↑/↓` or `j/k` to navigate, `Enter` to open, `r` to refresh manually, `a` to archive, `d` to delete
- Press `Tab` to view archived scans

### Archive List

- Browse all archived scans (compressed .zip files)
- Shows: Archive Name, Size, Events, Vulns, Findings, Date Archived
- `u` to unarchive (restore), `d` to delete permanently
- Press `Tab`, `q`, or `Escape` to return to scan list

### Archive Management

Save disk space by compressing old scans into ZIP archives:

**Archiving a scan:**
1. From the scan list, navigate to the scan you want to archive
2. Press `a` to archive
3. Confirm the operation
4. The scan folder is compressed to a .zip file and the original folder is deleted
5. Archive appears in the archive list (press `Tab` to view)

**Restoring an archive:**
1. Press `Tab` to view the archive list
2. Navigate to the archive you want to restore
3. Press `u` to unarchive
4. Confirm the operation
5. The archive is extracted and the .zip file is deleted
6. Press `q` to return to scan list and see the restored scan

**Safety features:**
- Cannot archive RUNNING scans
- Archive integrity is verified before deleting source folder
- Extraction is verified before deleting archive
- All operations require confirmation
- If any step fails, the operation is rolled back safely

**Deleting scans/archives:**
- **From scan list**: Press `d` to permanently delete a scan folder
- **From archive list**: Press `d` to permanently delete an archive file
- Cannot delete RUNNING scans
- Requires confirmation (action is permanent and cannot be undone)
- All scan data will be lost

## Work Tracking & Annotations

Track your security workflow by annotating vulnerabilities and findings with status, priority, and notes.

**How it works:**
- Annotations are stored in `.bbot_ui_annotations.json` alongside each scan
- References events by UUID - never modifies BBOT's original `output.json`
- Included automatically in archives for backup/restore
- Survives re-scans of the same target

**Annotating a vulnerability/finding:**
1. Navigate to the Vulnerabilities or Findings tab
2. Select an item (arrow keys or j/k)
3. Press `t` to open annotation dialog
4. Set status, priority (optional), and notes
5. Click Save or press Enter

**Quick shortcuts:**
- Press `x` to mark selected item as **False Positive**
- Press `i` to mark selected item as **Accepted Risk**
- These preserve existing priority and notes while updating status

**Status options:**
- 🆕 **New** - Default status for unannotated items
- 🔍 **Investigating** - Currently analyzing
- ✓ **Confirmed** - Verified as real issue
- ✗ **False Positive** - Not a real vulnerability
- 📢 **Reported** - Submitted to security team
- 🔧 **Fixed** - Issue has been resolved
- ⚠ **Accepted Risk** - Known but accepted

**Priority levels (optional):**
- 🔴 **Critical** - Requires immediate attention
- 🟠 **High** - Important, address soon
- 🟡 **Medium** - Normal priority
- 🟢 **Low** - Minor issue

**Features:**
- Status and Priority columns in Vulnerabilities/Findings tables
- Status filter dropdown - filter by specific status or "Actionable" items (default)
- Quick keyboard shortcuts (x/i) for rapid triage
- Workflow status charts in Statistics tab
- Notes field for detailed context
- Clear annotation button to reset
- Annotations persist across sessions and archives

**Status Filtering:**
- **Actionable** (default) - Shows only items needing attention (new, investigating, confirmed, reported)
- **All** - Shows all vulnerabilities/findings regardless of status
- **Specific statuses** - Filter by individual status (false-positive, fixed, etc.)
- Filter automatically updates when marking items with keyboard shortcuts

### Scan Viewer Tabs

- **Status Bar**: Shows scan status with real-time event count
  - **● RUNNING** (green) - Actively updating with new events
  - **✓ FINISHED** (blue) - Scan completed, no more updates
  - **⚠ INTERRUPTED** (yellow) - Scan was stopped/interrupted
- **Auto-refresh**: All tabs update every 2 seconds when scan is RUNNING
- **Smart detection**: Automatically stops polling FINISHED and INTERRUPTED scans
- Press `r` to refresh manually and see notification with new event count

**1. Vulnerabilities** - VULNERABILITY events sorted by severity (CRITICAL→HIGH→MEDIUM→LOW→INFO→UNKNOWN), with status, priority, and annotations (live updates)
**2. Findings** - FINDING events with status, priority, and annotations (live updates)
**3. Events** - All events with type filter, scope distance filter, multi-term search, and JSON details (live updates)
**4. Tree** - Two view modes (live updates):
   - **Discovery**: Shows how events were found through scan modules (parent-child relationships)
   - **Topology**: Logical network hierarchy (IP_RANGE → IP → OPEN_TCP_PORT)
**5. Statistics** - Event distribution, top 15 modules (ranked), scope distance charts, workflow status, and priority distribution (live updates)
**6. Configuration** - Syntax-highlighted preset.yml

## Multi-term Search

The Events tab supports powerful multi-term search:

- **Space-separated terms**: Use spaces to search for multiple terms (e.g., `httpx in-scope`)
- **AND logic**: Events must match ALL terms to appear in results
- **Fields searched**: data, type, module, host, tags, discovery_context
- **Combine with filters**: Works together with Type and Scope distance filters

Examples:
- `httpx in-scope` - Events from httpx module with in-scope tag
- `k11h HIGH` - Events related to k11h.de with HIGH severity
- `nuclei VULNERABILITY` - Vulnerabilities discovered by nuclei module

## Keyboard Shortcuts

**Navigation**: `↑/↓` or `j/k` | **Annotate**: `t` | **False Positive**: `x` | **Accepted Risk**: `i` | **View archives**: `Tab` (from scan list) | **Search**: `f` | **Refresh**: `r` | **Archive**: `a` (scan list) | **Unarchive**: `u` (archive list) | **Delete**: `d` | **Adjust split**: `←/→` | **Back/Quit**: `q` or `Escape`

## Live Refresh & Status Detection

bbot-ui automatically detects and displays updates from running scans in real-time:

### Smart Status Detection

The UI uses a **multi-method detection chain** to accurately determine scan status:

1. **SCAN Event Analysis**: Reads the last SCAN event's `status` field from `output.json`
   - `"FINISHED"` → Scan completed (has `finished_at` and `duration` fields)
   - `"RUNNING"` → Verify if actually running (proceed to step 2)

2. **Active Process Detection** (for RUNNING status):
   - **psutil** (auto-installed, cross-platform) - Checks if any process has `output.json` open

3. **Final Status**:
   - **RUNNING**: SCAN event says RUNNING + process actively has file open
   - **INTERRUPTED**: SCAN event says RUNNING + no process has file open (scan was Ctrl+C'd)
   - **FINISHED**: SCAN event says FINISHED (has completion data)

### Features
- **Accurate detection**: Immediately identifies interrupted scans without waiting for timeout
- **Performance optimized**:
  - Caches process checks for 5 seconds (avoids scanning all processes repeatedly)
  - Only checks RUNNING scans (skips expensive checks for FINISHED scans)
  - Smart polling stops checking FINISHED and INTERRUPTED scans
- **Incremental loading**: Efficiently reads only new events from `output.json`
- **Non-blocking**: UI remains fully responsive during updates
- **Cursor preservation**: Maintains your position in tables during refresh
- **Graceful handling**: Skips incomplete/malformed JSON lines from running scans
- **Configurable intervals**: Customize refresh rates for your needs

### Configuration

You can customize the live refresh behavior:

**Via command-line:**
```bash
./bbot-ui --scan-interval 5 --list-interval 10
```

**Defaults:**
- Scan detail view refreshes every **2 seconds**
- Scan list view refreshes every **3 seconds**

**Use cases:**
- **Fast networks/local scans**: Use shorter intervals (e.g., `--scan-interval 1`)
- **Remote/slow systems**: Use longer intervals (e.g., `--scan-interval 5`)
- **Reduce CPU usage**: Increase all intervals for less frequent checks

Settings are saved to `~/.bbot_ui_config.json` and persist across sessions.

## Troubleshooting

**Setup didn't complete properly?**
```bash
rm -rf ~/.bbot_ui_venv && ./bbot-ui
```

**Warning about psutil not installed?**
If you see a warning that psutil is missing, your venv is from an older version. Reinstall:
```bash
rm -rf ~/.bbot_ui_venv && ./bbot-ui
```

**No output.json found?**
Ensure scan directory contains `output.json` (BBOT generates this automatically)

**Python not found?**
```bash
# Ubuntu/Debian
sudo apt install python3 python3-venv

# macOS
brew install python3
```

## Requirements

- Python 3.8+
- Auto-installs: textual>=0.47.0, rich>=13.0.0, psutil>=5.9.0

**Note**: psutil is used for accurate scan status detection by checking if any process has the scan file open.

## Performance

The UI is optimized for large scans:

**Display Limits:**
- **Vulnerabilities tab**: 1000 rows max (sorted by severity)
- **Findings tab**: 1000 rows max
- **Events tab**: 1000 rows max (use filters for large scans)
- **Tree views**: 500 nodes max (use filters to focus on specific areas)
- **Scan list**: Fast loading using sampling for large scans (>1MB)

**File Reading:**
- Reads from both ends of file to find SCAN events (handles reused scan directories)
- Detects most recent scan by timestamp (supports multiple runs in same directory)
- Estimates event counts for large scans using file size and sampling
- Caches status checks to avoid repeated process scans

**Auto-Refresh:**
- Timer automatically stops for FINISHED/INTERRUPTED scans
- Only checks RUNNING scans for updates
- Results cached for 5 seconds

## Tips

- Use filters (type, scope distance) to focus on specific events in large scans
- Event counts for large scans (>1MB) are estimates for performance
- Delete `~/.bbot_ui_venv/` to force clean reinstall

## License

MIT
