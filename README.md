# Clockin

A native time tracker that shows what your work is earning while you do it. Clock in, pause, clock out, and Clockin keeps the hours, the money and the history, including timecards imported from a CSV or pasted from the timecard page.

The repository holds two apps that share the same data model:

| App | Where | Requires |
|---|---|---|
| **Mac** | repository root (`Sources/Clockin`) | macOS 14, Swift 6 |
| **iPhone** | [`iOS/`](iOS/README.md) | iOS 17, Xcode |

Each app keeps its own data on its own device. They do not sync.

## What both apps do

**Tracking.** Clock in, pause and resume, clock out or cancel without saving. Start with elapsed time when you forgot to press the button, add a past entry by hand, and edit or delete any session.

**Earnings.** Hourly rates follow a schedule with effective dates, so a raise applies from its day onward and older sessions keep their rate. USD earnings are shown with their Turkish lira equivalent, using the current rate for today and each day's historical rate in history. Rates come from the free, keyless [Frankfurter API](https://frankfurter.dev/) and are cached locally.

**History.** Earnings history with 7D, 30D, 3M and ALL periods, a daily chart, and per-day details.

**Timecard import.**
- CSV files with the `Start Time`, `End Time`, `Duration`, `Notes` and `Time Sheet Source` columns.
- Approved entries copied from the timecard page and pasted in, as one row or a full page. The paste preview reads the page's Approved total and warns when the copied rows cover only part of it.
- Every import is reviewed before it is applied. Identical rows are skipped. A row that overlaps a timer entry, or an entry from the same timecard, on the same day by at least half of the shorter one corrects that entry instead of adding a second copy.

**Progress.** Daily and monthly goals, a work heatmap by day, week or month, reports and records, a level with streaks and 46 badges, and a shareable stats image.

**Focus.** An animated companion, a focus chime at a chosen interval of worked time, and a focus radio.

**Themes.** Eight themes, each with its own typography: Carbon, Neon Orange, Electric Blue, Synthwave, Data Dense, Aurora, Terminal Amber and Daylight.

**Backups.** An automatic backup at most once a day, the last 30 kept, plus manual export and restore.

## Mac app

Clockin lives in the menu bar. Double-clicking the app opens its window; closing the window keeps the menu bar timer running.

- **Window:** Timer, History, Heatmap, Progress and Settings tabs, with interface sizes from 100% to 150%.
- **Menu bar:** the running timer and today's totals. Minimal mode hides the window and lets you choose which fields the menu bar shows.
- **Pinned window:** an always-visible timer, resizable from its edges, in five layouts: Compact, Money (live earnings and per-second momentum), Goal, All and Total.
- **Keyboard shortcuts** that work while Clockin runs:

  | Shortcut | Action |
  |---|---|
  | ⌥⌘I | Clock in, or resume a paused session |
  | ⌥⌘P | Pause or resume |
  | ⌥⌘O | Clock out |
  | ⌥⌘E | Open the Clockin window |

- **Focus chime:** off by default; every 1 to 120 minutes of worked time, with a choice of Glass, Ping, Pop, Tink, Funk, Submarine or Sosumi and its own volume.
- **Updates:** Settings can check GitHub for newer commits, automatically at most every six hours.

### Build and run

```bash
chmod +x build-app.sh
./build-app.sh
open dist/Clockin.app
```

Data is stored at:

```text
~/Library/Application Support/Clockin/clockin.json
```

When no rate schedule exists yet, the stored hourly rate becomes the first rule, effective July 1, 2026. Add an earlier rule to change the rate of older sessions.

### Checks

Dependency-free checks, run from the repository root. Each exits non-zero on the first failure.

```bash
# models, CSV and pasted timecards, currency formatting
swiftc Sources/Clockin/Models.swift Sources/Clockin/CSVImporter.swift Sources/Clockin/PastedTextImporter.swift Tests/manual/main.swift -o /tmp/clockin-tests && /tmp/clockin-tests

# editing a session, unreadable data file
swiftc Sources/Clockin/Models.swift Sources/Clockin/ClockStore.swift Sources/Clockin/CSVImporter.swift Sources/Clockin/PastedTextImporter.swift Sources/Clockin/ImportComparison.swift Tests/manual/store/main.swift -o /tmp/clockin-store-tests && /tmp/clockin-store-tests

# failed saves are reported, not shown as success
swiftc Sources/Clockin/Models.swift Sources/Clockin/ClockStore.swift Sources/Clockin/CSVImporter.swift Sources/Clockin/PastedTextImporter.swift Sources/Clockin/ImportComparison.swift Tests/manual/save/main.swift -o /tmp/clockin-save-tests && /tmp/clockin-save-tests

# re-importing a corrected timecard, and the cases that must stay separate
swiftc Sources/Clockin/Models.swift Sources/Clockin/ClockStore.swift Sources/Clockin/CSVImporter.swift Sources/Clockin/PastedTextImporter.swift Sources/Clockin/ImportComparison.swift Tests/manual/reimport/main.swift -o /tmp/clockin-reimport-tests && /tmp/clockin-reimport-tests
```

The optional live exchange rate check needs a network connection:

```bash
swiftc -parse-as-library Sources/Clockin/ExchangeRates.swift Tests/manual/exchange.swift -o /tmp/clockin-exchange-test && /tmp/clockin-exchange-test
```

## iPhone app

The same tracker with what a phone adds: home and lock screen widgets, a Live Activity with pause and clock out on the lock screen and in the Dynamic Island, and Shortcuts actions for clocking in, out and pausing.

Screens are Today, History, Insights and Badges, with Settings behind the gear on Today. Data lives in an App Group on the phone so the widgets can read it.

A few things differ from the Mac on purpose. The level counts only hours worked and streaks, never goals, so it cannot be raised by lowering a goal. The focus chime uses notifications, since an iPhone app is usually suspended.

Building, the project layout and the iPhone checks are in [`iOS/README.md`](iOS/README.md). [`iOS/PARITY.md`](iOS/PARITY.md) compares the two apps feature by feature.
