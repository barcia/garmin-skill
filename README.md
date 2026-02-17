# Garmin Skill

Agent skill for creating and uploading workouts and body composition data to Garmin Connect.

Designed to be used with AI coding agents (Claude Code, Cursor, etc.) as a [skill](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/tutorials#create-custom-slash-commands). The agent reads `SKILL.md`, follows the instructions, and uses the included Python scripts to generate Garmin-compatible JSON payloads.

## What It Does

| Capability | Description |
|------------|-------------|
| **Simple runs** | Create time or distance-based workouts with pace/HR targets |
| **Interval runs** | Create structured interval workouts with warmup, intervals, recovery, and cooldown |
| **Schedule workouts** | Assign workouts to specific dates in the Garmin calendar |
| **Body composition** | Upload weight, body fat %, muscle mass, and other metrics |
| **Eufy import** | Convert Eufy smart scale CSV exports to Garmin-compatible format |

## Requirements

- **Python 3.12+** (scripts are standalone, no extra dependencies)
- **[Garmin MCP](https://github.com/Taxuspt/garmin_mcp)** — MCP server that handles authentication and communication with the Garmin Connect API (powered by [python-garminconnect](https://github.com/cyberjunky/python-garminconnect))

## Setup

### 1. Install the Garmin MCP server

Add to your MCP client configuration (Claude Desktop, Cursor, etc.):

```json
{
  "mcpServers": {
    "Garmin MCP": {
      "command": "uvx",
      "args": [
        "--python", "3.12",
        "--from", "git+https://github.com/Taxuspt/garmin_mcp",
        "garmin-mcp"
      ]
    }
  }
}
```

On first run, the Garmin MCP will prompt for Garmin Connect credentials and store tokens locally.

### 2. Install the skill

Clone the repo and register `SKILL.md` as a skill in your agent:

```bash
git clone git@github.com:barcia/garmin-skill.git
```

The skill path to register is `SKILL.md`.

## How It Works

The workflow is always:

```
Python script generates JSON → Garmin MCP uploads to Garmin Connect → (optionally) schedule on calendar
```

The agent reads the SKILL.md decision table, picks the right script, runs it with the appropriate arguments, and uses the Garmin MCP tools to upload the result.

### Workouts

**Simple run** (time or distance, with optional pace/HR targets):

```bash
python scripts/simple-run.py \
  --title "Easy Run" \
  --duration 30 --duration-type time \
  --target pace --pace-from 5.30 --pace-to 6.00
```

**Interval run** (structured repeats with per-phase targets):

```bash
python scripts/interval-run.py \
  --title "6x1min Z4" \
  --warmup-duration 10 \
  --reps 6 \
  --interval-duration 1 --interval-type time \
  --interval-target hr --interval-target-value 4 \
  --recovery-duration 1 \
  --cooldown-duration 5
```

Both scripts output a JSON payload that the agent uploads via:

```
Garmin MCP → upload_workout → workoutId → schedule_workout (optional)
```

### Body Composition

**Manual measurement:**

```bash
python scripts/upload_body_composition.py \
  --date 2026-02-03 --weight 75.5 --percent-fat 18.5 --muscle-mass 35.2
```

**Import from Eufy smart scale:**

```bash
python scripts/eufy_to_json.py eufy_export.csv --latest 5
```

## Project Structure

```
├── SKILL.md                          # Agent skill definition (entry point)
├── scripts/
│   ├── simple-run.py                 # Simple run workout generator
│   ├── interval-run.py               # Interval workout generator
│   ├── upload_body_composition.py    # Body composition JSON generator
│   └── eufy_to_json.py              # Eufy CSV → Garmin JSON converter
└── references/
    ├── workouts.md                   # Garmin workout JSON schema docs
    ├── fit.md                        # FIT protocol, scales, physique ratings
    └── eufy-to-garmin.md            # Eufy export format and field mapping
```

## Acknowledgments

- [python-garminconnect](https://github.com/cyberjunky/python-garminconnect) — Python API wrapper for Garmin Connect that powers the underlying MCP server
- [Garmin MCP](https://github.com/Taxuspt/garmin_mcp) — MCP server for Garmin Connect

## License

[GPL-3.0](LICENSE)
