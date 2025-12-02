# CLAUDE.md - BBOT TUI Developer Guide

This document provides a comprehensive overview of the BBOT TUI codebase for AI assistants and developers picking up work on this project.

## Project Overview

**BBOT TUI** is a self-contained terminal user interface for browsing and analyzing [BBOT](https://www.blacklanternsecurity.com/bbot/) security scan results. It's a single Python file that automatically sets up its own virtual environment and dependencies on first run.

### Key Features
- 🚀 Zero setup - single self-installing executable
- 🔴 Live refresh with accurate scan status detection
- 📦 Archive management (compress/restore scans)
- 🔍 Separate views for vulnerabilities and findings
- 🌳 Discovery tree showing event relationships
- 📊 Rich statistics and filtering
- ⚡ Optimized for large scans (1000s of events)

### Tech Stack
- **Textual** (>=0.47.0) - TUI framework
- **Rich** (>=13.0.0) - Text formatting
- **psutil** (>=5.9.0) - Process detection (cross-platform)
- Python 3.8+

## File Structure

```
bbot-ui/
├── bbot-ui              # Main executable (single file application)
├── README.md            # User documentation
├── LICENSE              # MIT license
├── docs/
│   └── demo.gif        # Demo screenshot
└── CLAUDE.md           # This file (developer guide)
```

## Architecture Overview

The application is a **single-file Python script** (`bbot-ui`) with two main sections:

### 1. Bootstrap Section (Lines 1-109)
- Virtual environment setup
- Dependency installation
- Auto-re-execution in venv context

### 2. Main Application (Lines 110+)

#### Core Components

```
BBotViewer (App)
├── ScanListScreen          # Main screen - browse scans
│   ├── Archive functionality ('a' key)
│   ├── Delete functionality ('d' key)
│   └── Switch to archives (Tab key)
│
├── ArchivesListScreen      # Archive browser
│   ├── Unarchive functionality ('u' key)
│   ├── Delete functionality ('d' key)
│   └── Return to scans (Tab/q/Escape)
│
├── ScanViewerScreen        # Individual scan viewer
│   ├── VulnerabilitiesTable
│   ├── FindingsTable
│   ├── EventsTable
│   ├── DiscoveryTree
│   ├── StatisticsView
│   └── PresetView
│
└── ConfirmDialog           # Modal confirmation dialogs
```

## Key Classes and Responsibilities

### Helper Functions (Lines 148-350)

#### `create_archive(scan_path: Path) -> tuple[bool, str]`
- Creates ZIP archive from scan folder
- Verifies integrity with `zipfile.testzip()`
- Deletes source folder only after verification succeeds
- **Safety:** Atomic operation, rollback on failure

#### `extract_archive(archive_path: Path) -> tuple[bool, str]`
- Extracts ZIP archive to folder
- Validates output.json exists and is readable
- Deletes archive only after verification succeeds
- **Safety:** Atomic operation, rollback on failure

#### `get_archive_stats(archive_path: Path) -> dict`
- Reads statistics from archive without extracting
- For files <1MB: counts exactly
- For files >=1MB: estimates using 100KB sample + ratio
- Returns: size, date, event_count, vuln_count, finding_count

### Core Classes

#### `ScanData` (Lines ~353-700)
Parses and manages BBOT scan data from output.json files.

**Key Methods:**
- `_find_latest_scan_event()` - Handles reused scan directories
- `_load_events()` - Incremental loading for large files
- `get_scan_status()` - Determines RUNNING/FINISHED/INTERRUPTED

#### `ScanListScreen` (Lines ~1276-1896)
Main screen showing all scans in a table.

**Key Features:**
- Auto-refresh every 3 seconds (configurable)
- Status detection using psutil (checks if output.json is open)
- Archive functionality ('a' key)
- Delete functionality ('d' key)
- Switch to archives (Tab key)
- **Important:** `on_screen_resume()` refreshes when returning from archives

**Status Detection Chain:**
1. Read last SCAN event from output.json
2. If status="RUNNING", check if process has file open (psutil)
3. Result: RUNNING (file open) | INTERRUPTED (not open) | FINISHED

**Performance Optimizations:**
- Caches process checks for 5 seconds
- Only checks RUNNING scans
- Estimates event counts for large files (>1MB)

#### `ArchivesListScreen` (Lines ~1898-2122)
Browse archived scans (ZIP files).

**Key Features:**
- Lists all .zip files in scan directory
- Shows size, date, event/vuln/finding counts
- Unarchive functionality ('u' key)
- Delete functionality ('d' key)
- Return to scans (Tab/q/Escape)

#### `ScanViewerScreen` (Lines ~2015-2647)
Detailed view of a single scan with tabs.

**Tabs:**
1. **Vulnerabilities** - VULNERABILITY events, sorted by severity
2. **Findings** - FINDING events
3. **Events** - All events with filters and search
4. **Tree** - Discovery (parent-child) or Topology view
5. **Statistics** - Event distribution and module rankings
6. **Configuration** - preset.yml syntax highlighted

**Live Refresh:**
- Auto-updates every 2 seconds when scan is RUNNING
- Stops polling for FINISHED/INTERRUPTED scans
- Preserves cursor position during updates

#### `ConfirmDialog` (Lines ~2124-2150)
Modal confirmation for destructive operations.

**Usage Pattern:**
```python
self.app.push_screen(
    ConfirmDialog(
        "Message with warning",
        lambda confirmed: self._handle_confirmation(confirmed, data)
    )
)
```

## Recent Changes (December 2, 2025)

### Archive Management System
**Added:**
- Archive scans to ZIP format (compress and delete original)
- Unarchive scans (extract and delete ZIP)
- Delete functionality for both scans and archives
- Tab key navigation between scan list and archive list
- Auto-refresh when returning from archive view
- Accurate event counting in archives (exact for <1MB, estimated for >=1MB)

**Safety Features:**
- Cannot archive/delete RUNNING scans
- Integrity verification before deletion
- Confirmation dialogs for all destructive operations
- Atomic operations with rollback on failure

### UI Flow Changes
- **Old:** Single scan list screen
- **New:** Two screens with Tab navigation
  - ScanListScreen ⟷ ArchivesListScreen (Tab key toggles)
  - Press 'q' from archive list also returns to scans

### Code Cleanup
- Removed unused severity filter functionality
- Removed failed widget/tab approach artifacts
- Cleaned up unused CSS rules and imports
- Removed: `TabbedContent`, `TabPane`, `Select` imports
- Removed: MainScreen, ScanListWidget, ArchivesListWidget classes

## Important Implementation Patterns

### 1. Screen Lifecycle
```python
def on_mount(self) -> None:
    """Called when screen is first mounted"""
    # Setup tables, start timers

def on_unmount(self) -> None:
    """Called when screen is removed from stack"""
    # Cleanup timers, stop processes

def on_screen_resume(self) -> None:
    """Called when returning to screen (after pop_screen)"""
    # Refresh data, update UI
```

### 2. Confirmation Pattern
```python
# Show dialog
self.app.push_screen(
    ConfirmDialog(message, callback)
)

# Handle response
def _handle_confirmation(self, confirmed: bool, data: Dict) -> None:
    if not confirmed:
        self.notify("Cancelled")
        return
    # Perform action
```

### 3. Table Refresh Pattern
```python
def refresh_table(self) -> None:
    table = self.query_one("#table-id", DataTable)
    old_cursor = table.cursor_row  # Preserve cursor position

    table.clear()
    # Rebuild rows

    if old_cursor < table.row_count:
        table.move_cursor(row=old_cursor)
```

### 4. Rich Text in Tables
To prevent markup interpretation (e.g., `[brackets]`):
```python
from rich.text import Text
description = Text(raw_string, no_wrap=True, overflow="ellipsis")
table.add_row(description)  # Not a string!
```

### 5. Auto-Refresh Timers
```python
# Setup
self.refresh_timer = self.set_interval(interval, self.check_updates)

# Cleanup
def on_unmount(self) -> None:
    if self.refresh_timer:
        self.refresh_timer.stop()
```

## Configuration

### User Config File
Location: `~/.bbot_ui_config.json`

```json
{
  "theme": "textual-dark",
  "scan_refresh_interval": 2.0,
  "list_refresh_interval": 3.0
}
```

### Virtual Environment
Location: `~/.bbot_ui_venv/`

**Marker file:** `.setup_complete` indicates setup is done

**To force reinstall:**
```bash
rm -rf ~/.bbot_ui_venv && ./bbot-ui
```

## Performance Considerations

### Display Limits
- Vulnerabilities table: 1000 rows max
- Findings table: 1000 rows max
- Events table: 1000 rows max
- Tree views: 500 nodes max

### Large File Handling
- Files >1MB: Event counts are estimated via sampling
- Process checks: Cached for 5 seconds
- Only RUNNING scans checked for process status
- Incremental event loading for scan viewer

### Archive Statistics
- Files <1MB: Count exactly
- Files >=1MB: Sample 100KB, estimate by ratio
- Result cached until refresh

## Common Development Tasks

### Adding a New Keybinding
```python
BINDINGS = [
    Binding("x", "my_action", "Description"),
]

def action_my_action(self) -> None:
    """Called when 'x' is pressed"""
    # Implementation
```

### Adding a New Table Column
```python
def on_mount(self) -> None:
    table = self.query_one("#table-id", DataTable)
    table.add_columns("Col1", "Col2", "NewCol")  # Add here

def refresh_table(self) -> None:
    # Add data to new column
    table.add_row(col1_val, col2_val, new_col_val)
```

### Adding a New Screen
```python
class MyNewScreen(Screen):
    BINDINGS = [
        Binding("q", "quit", "Back"),
    ]

    def compose(self) -> ComposeResult:
        yield Header()
        yield Static("Content")
        yield Footer()

    def action_quit(self) -> None:
        self.app.pop_screen()

# Navigate to it
self.app.push_screen(MyNewScreen())
```

## Testing Checklist

When making changes, verify:
- [ ] Scan list displays correctly
- [ ] Archive list displays correctly
- [ ] Tab navigation works both ways
- [ ] Archive/unarchive operations work
- [ ] Delete operations work (with confirmation)
- [ ] RUNNING scans cannot be archived/deleted
- [ ] Auto-refresh works correctly
- [ ] Cursor position preserved during refresh
- [ ] Status detection accurate (RUNNING/FINISHED/INTERRUPTED)
- [ ] Large files (>1MB) load efficiently
- [ ] Event counts accurate in both views

## Known Limitations

1. **Archive counts are estimates** for files >1MB (acceptable tradeoff)
2. **Display limits** prevent showing all events in very large scans
3. **Process detection** requires psutil (bundled as dependency)
4. **Single scan directory** - doesn't support multiple BBOT installations

## Future Enhancement Ideas

- [ ] Multi-select for batch archive/delete operations
- [ ] Export filtered results to CSV/JSON
- [ ] Scan comparison view (diff two scans)
- [ ] Advanced filtering (regex, date ranges)
- [ ] Scan scheduling/automation
- [ ] Integration with BBOT CLI commands
- [ ] Dark/light theme toggle in UI
- [ ] Configurable keyboard shortcuts
- [ ] Plugin system for custom views

## Debugging Tips

### Enable Textual DevTools
```bash
textual console  # In one terminal
./bbot-ui       # In another terminal (press Ctrl+\ to see logs)
```

### Common Issues

**Empty scan list:**
- Check if `output.json` exists in scan folders
- Verify permissions on scan directory
- Check for errors in textual console

**Status detection incorrect:**
- Verify psutil is installed: `~/.bbot_ui_venv/bin/pip list | grep psutil`
- Check if bbot process has permissions to open files

**Archive operations fail:**
- Verify ZIP file integrity: `unzip -t archive.zip`
- Check disk space for extraction
- Verify write permissions in scan directory

## Git Workflow

```bash
# Current state (as of Dec 2, 2025)
git status
# On branch main
# All changes committed

# Create feature branch
git checkout -b feature/your-feature

# Make changes, commit
git add bbot-ui
git commit -m "feat: description"

# Update README if needed
git add README.md
git commit -m "docs: update README"
```

## Contact & Resources

- **BBOT Project:** https://www.blacklanternsecurity.com/bbot/
- **Textual Docs:** https://textual.textualize.io/
- **Rich Docs:** https://rich.readthedocs.io/

---

*Last Updated: December 2, 2025*
*Version: 1.0 with Archive Management*
