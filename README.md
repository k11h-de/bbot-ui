# BBOT TUI Viewer

A self-contained terminal UI for browsing and analyzing BBOT scan results.

## Features

- 🚀 **Zero Setup** - Single self-installing file, no manual dependencies
- 📋 **Scan Browser** - Navigate multiple scans with findings count
- 🔍 **Findings View** - Dedicated tab for vulnerabilities and findings
- 🌳 **Discovery Tree** - Hierarchical view showing parent-child event relationships
- 📊 **Rich Statistics** - Beautiful tables with event distribution and scope analysis
- 🔎 **Event Explorer** - Filter, search, and inspect all scan events
- ⚙️ **Config Viewer** - View preset.yml configuration

## Quick Start

```bash
# Copy to server and run (auto-installs on first run)
./bbot-viewer

# Or specify custom path
./bbot-viewer /path/to/scans
```

First run creates `.bbot_viewer_venv/` and installs dependencies. Subsequent runs launch instantly.

## Usage

```bash
./bbot-viewer                          # Default: ~/.bbot/scans
./bbot-viewer /path/to/scans          # Browse all scans in directory
./bbot-viewer ~/.bbot/scans/scan-name # View specific scan
```

## Interface

### Scan List
- Browse all scans with event/findings counts
- ⚠ indicator shows scans with vulnerabilities
- `↑/↓` or `j/k` to navigate, `Enter` to open

### Scan Viewer Tabs

**1. Findings** - Vulnerabilities and findings with severity, host, description
**2. Events** - All events with type filter, scope distance filter, multi-term search, and JSON details
**3. Tree** - Two view modes:
   - **Discovery**: Shows how events were found through scan modules (parent-child relationships)
   - **Topology**: Logical network hierarchy (IP_RANGE → IP → OPEN_TCP_PORT)
**4. Statistics** - Event distribution, top 15 modules (ranked), scope distance charts
**5. Configuration** - Syntax-highlighted preset.yml

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

**Navigation**: `↑/↓` or `j/k` | **Switch tabs**: `Tab` | **Search**: `f` | **Adjust split**: `←/→` | **Back/Quit**: `q` or `Escape`

## Troubleshooting

**Setup didn't complete properly?**
```bash
rm -rf .bbot_viewer_venv && ./bbot-viewer
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
- Auto-installs: textual>=0.47.0, rich>=13.0.0

## Tips

- Viewer limits display to 1000 events; use filters for large scans
- Delete `.bbot_viewer_venv/` to force clean reinstall
- Each directory maintains its own venv when script is copied

## License

MIT
