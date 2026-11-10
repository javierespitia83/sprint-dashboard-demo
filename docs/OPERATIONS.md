# OPERATIONS.md — running the dashboard against the real tracker

This is the operational runbook from the original team deployment. Paths use `$AF_SNAPLOGIC_ROOT`, the project folder that held the tracker workbook, the exporter and the agent skills. The demo in this repository already ships with anonymized JSON, so none of this is needed to run it.

## How to refresh the data

The sprint page reads `sprints.json` and only the selected snapshot it names; the backlog page reads `backlog.json`; Quality Intake reads `quality-intake.json`. The pages never read Excel. Closed-sprint snapshots are frozen at sprint rotation (`DATA_CONTRACT.md`, "Sprint rotation and manifest").

1. Update the tracker (`$AF_SNAPLOGIC_ROOT/03_Tracker/daily_tracker.xlsx`) and **close it in Excel** (the exporter refuses to run while the lock file `~$daily_tracker.xlsx` exists).
2. From `$AF_SNAPLOGIC_ROOT`, run:
   ```bash
   python3 05_Automation/export_dashboard_snapshots.py
   ```
3. Reload the page in the browser.

The export is manual and read-only with respect to the tracker. It writes the active sprint's snapshot (`snapshots/sprint-<id>.json`, the file of its `sprints.json` entry), `backlog.json` and `quality-intake.json`. Full behaviour: the exporter skill listed under "Paths".

## How to test

Run the automated acceptance checks from this folder when verification is requested:

```bash
node tests/acceptance/run_acceptance.js          # about two minutes
node --test tests/unit.test.js        # unit tests only, about a second (the line above runs them too)
```

They need Node 22 or newer, python3 and Google Chrome (set `CHROME_PATH` if Chrome is not in the default macOS location). The acceptance run prints PASS, FAIL or SKIP per check and exits **0** when every check passes, **1** when a check fails and **2** when the harness itself fails; the results and screenshots are kept in the scratch folder unless the run is clean. It is read-only: it serves a staged copy of only the app and data files, and proves by content hash that nothing in this folder changed. It also checks that the tracker workbook is unchanged when it can find it (default `$AF_SNAPLOGIC_ROOT/03_Tracker/daily_tracker.xlsx`, falling back to the former `03_Team_Enablement/` location; set `AF_TRACKER_PATH` to point elsewhere; without a tracker that check is skipped). Details: `ACCEPTANCE.md` and `IMPLEMENTATION.md` §10.

