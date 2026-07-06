# mercury -- display

## Problem

Structured data (list of records, tree of files, stream of log lines) must be shown on a screen (terminal, web page, GUI, embedded display). The same data may be shown in different ways depending on the screen type.

## Input

- A list of items to display
- A viewport position (scroll offset) and size (rows x columns)

## Output

- Rendered output to the screen

## Observable Behavior

- Given a list and a viewport, produce visible output for the current viewport slice
- When the viewport position changes, update the visible output
- When the data changes (items added, removed, modified), reflect those changes in the display

## Minimal Code

```
function display_list(items, page=0, page_size=20):
    start = page * page_size
    end = start + page_size
    for item in items[start:end]:
        print(item)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Flicker | Screen clears and redraws on every update | Double-buffer or use incremental updates |
| Scroll starvation | Rendering a large dataset blocks scrolling | Virtual scrolling: render only visible rows |
| Layout cost | 100,000 items all compute layout even though only 30 are visible | Virtual scrolling with fixed row heights |
| Input lag | Keystroke processing lags during slow render | Render on separate thread from input |
| Line wrap break | Resizing the window clips or breaks content | Re-layout on resize with debounce |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Virtual vs. full rendering | Virtual renders only visible items, scales to millions. Full renders everything, simpler but limited by memory. |
| Immediate vs. retained | Immediate redraws everything each frame. Retained draws once, updates on change. |
| Text vs. graphics | Text (terminal, HTML) is simple but limited layout. Graphics (canvas, OpenGL) enables rich visuals but more code. |

## Start With

The minimal code above. Add pagination before virtual scrolling. Add input handling (arrow keys, mouse) after basic display works.

## Split Into Its Own Module When

- More than one display type is needed (terminal + GUI + web).
- Virtual scrolling is needed for datasets larger than 10,000 items.
- User interaction (selection, scroll, search) is required.
- The display must support themes, custom layouts, or accessibility features.

## Signs This Is Present

- Code iterates and prints or renders items to a screen.
- Scroll position, pagination, or viewport state is tracked.
- A UI framework or library is used to present data.
- User input (keyboard, mouse) changes what is visible.

## Common Mistakes

- Rendering all items in a large dataset (memory exhaustion, slow startup).
- Not handling terminal/window resize (layout breaks or content clipped).
- Mixing display logic (where to show) with formatting logic (how to show).
- Blocking the input handler during a slow render (unresponsive UI).
- Not testing with empty dataset, one item, or maximum size.
