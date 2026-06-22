# ------------------------------------------------------------------------------
#  mercury-display.md
#  Data variation -- same rendering pipeline, different values to display.
# ------------------------------------------------------------------------------

# - Problem -

Your program has data that a human needs to see. You need to format it, keep it up to date, and show it on a screen, terminal, or web page.

# - When You'll See This -

Any time you build something that shows data to a person. Think CLI tools that print results, dashboards, text editors, games, or monitoring UIs. Display almost always pairs with a data source (file, database, computation result) and often leads into user commands.

# - How It Works -

- **Source**: where the data comes from -- could be a file, a live stream, or just a variable in memory.
- **Display surface**: where the output goes -- a terminal window, a GUI widget, an HTML element, or a file.
- **User**: the person reading the display. They might respond by clicking a button or typing a command.

# - What Must Always Be True -

- Every value shown on screen can be traced back to a real source value at some point in time.
- Format conversions keep the information the user actually needs (you can drop details they don't care about).
- Updates happen fast enough for the context -- under a second for interactive use, a few seconds for a dashboard.
- The screen never shows contradictory data from the same source at the same time.

# - What Goes Wrong -

- **Stale data**: the display shows old information because it hasn't refreshed.
- **Overflow**: there's more data than fits on screen, so some of it gets cut off silently.
- **Inconsistent views**: different parts of the UI update from different snapshots and show conflicting info.
- **Lost precision**: a conversion drops details the user actually needed (like rounding off cents in a price).
- **Slow updates**: the display refresh can't keep up with the data changes, causing flicker or missed states.
- **Misleading visuals**: a colour or icon implies something that isn't true in the data.

# - Design Trade-Offs -

- **Perfect rendering vs. speed**: showing every detail accurately takes time; sometimes a rough picture fast is better.
- **Everything vs. something useful**: showing all data at once overwhelms the user; hiding some behind navigation loses context.
- **Smooth vs. responsive**: smoothing out updates looks nicer but adds delay.

# - Related Problems -

- **mars/persistence**: data often comes from storage.
- **neptune/transformation**: raw data usually needs reshaping before display.
- **janus/reduction**: dashboards show summaries, not every row.
- **jupiter/validation**: user commands triggered by what they see need checking.
- **saturn/routing**: clicks and key presses need to reach the right handler.
- **earth/sorting**: ordered lists need sorting first.
- **venus/filtering**: showing a subset means filtering first.

# - Getting Started -

Print raw values to stdout with `print()` / `console.log()` / `println!`. Format as a table with simple string padding. Re-render everything on each update.

# - When to Split Into Its Own Module -

- You need multiple output targets (terminal, HTML file, JSON export).
- Data arrives faster than you can re-render -- time to add throttling or diffing.
- Users interact with the display (scroll, click, filter) -- events need to flow back to data.
- The display layout needs to survive a restart.

# - Signs You're in This Pattern -

- Your computation code is mixed with `print` / `format` / `write` calls.
- Changing the output format means editing the same functions that do the real work.
- You're copy-pasting rendering logic for different output targets.
- The same data flows through different display paths depending on context.

# - Common Mistakes -

- Building a fancy widget framework before you have more than one view type.
- Separating display from data fetching before you have a second data source.
- Implementing virtual scrolling before measuring that rendering everything is actually slow.

# - The Core Challenge -

Display has a timing problem. The source data changes on its own schedule, but the display updates on a different cycle. You have to decide when to sample the data, when to push updates, and what to show during transitions -- all without the user acting on stale or inconsistent information.
