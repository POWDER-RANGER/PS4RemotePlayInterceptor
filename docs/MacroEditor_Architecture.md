# GUI Macro Editor Architecture (Draft)

This document outlines the draft architecture and UX plan for the Phase 2 GUI Macro Editor for PS4/PS5 Remote Play Interceptor.

## 1) Overview: WPF/WinForms Layout

Target: Windows desktop (initial). Technology: WPF (preferred) with MVVM; WinForms acceptable fallback for rapid prototyping of timeline.

- Shell Window (MainWindow)
  - Top AppBar: File, Edit, View, Tools, Help
  - Left Panel: Macro Library (tree/list), Tags/Filter, Search
  - Center: Timeline Editor (tracks, grid, playhead), Zoom controls
  - Bottom: Transport Controls (Record, Play, Pause, Stop, Loop, FPS), Status Bar (latency, CPU, connection)
  - Right Panel (Dockable): Properties Inspector, Event Editor, Validation/Problems, Controller State Preview
  - Modal/Dialogs: New Macro Wizard, Import/Export, Preferences, Controller Binding Map

- MVVM Layers
  - View: XAML views for Shell, Timeline, Inspector, Library
  - ViewModels: ShellVM, MacroLibraryVM, TimelineVM, TrackVM, ClipVM, SelectionVM, PlaybackVM
  - Models: MacroDocument, Track, Clip(Event), Keyframe, Condition, Metadata
  - Services: SerializationService (.ps4macro), PlaybackEngine, RecordingService, ValidationService, UndoRedoService, StorageService, InterceptorBridge (IPC), TelemetryService

- Theming + Accessibility
  - Light/Dark themes, scalable fonts, high-contrast mode
  - Full keyboard navigation; screen reader labels; tooltips

## 2) Timeline Interface: Section Diagram

Tracks are time-aligned rows. Each clip contains events (button presses/axes) with durations. Grid snaps to tick rate (default 60 fps) but supports ms precision.

- Header: Time ruler (frames/ms), markers, loop region
- Track Types:
  - Button Track: Cross, Circle, Square, Triangle, L1, R1, L2, R2, L3, R3, Options, Share, PS
  - Axis Track: LX, LY, RX, RY, L2Analog, R2Analog
  - Composite Track: Custom groups/macros
  - Script Track: Lua/Python callback hooks at timestamps
- Interactions:
  - Drag to reposition clips; resize for duration
  - Multi-select, copy/paste/duplicate; ripple delete
  - Zoom, pan, snap (frame, 5ms, 1ms); nudge with arrows
  - Validation badges on clips (timing conflicts, out-of-range values)

ASCII diagram of the layout:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Time Ruler: 00:00.000 ┆ 00:00.500 ┆ 00:01.000 ┆ 00:01.500 ┆ 00:02.000       │
├──────────────────────────────────────────────────────────────────────────────┤
│ [Button: Cross]  ███──────█   ████────────────                                 │
│ [Button: Circle]     ████───────  ██                                            │
│ [Axis: LX]       ──/‾‾‾‾\────\__/
│ [Axis: LY]       ‾‾\____/‾‾‾‾‾‾‾\__                                           │
│ [Script: Lua]                 ◉ onComboStart()                                  │
├──────────────────────────────────────────────────────────────────────────────┤
│ Transport: ◼ ◄◄ ▶ ▌▌ ►► ⟲ | FPS:60 | Snap:Frame | Zoom: 100% | Playhead: 01.000│
└──────────────────────────────────────────────────────────────────────────────┘
```

## 3) .ps4macro JSON Serialization Schema (Draft)

Versioned, human-diffable JSON. Time is milliseconds by default; frames optional if fps provided.

```json
{
  "$schema": "https://powder-ranger.github.io/schemas/ps4macro.schema.json",
  "version": "1.0.0",
  "metadata": {
    "id": "guid-or-ulid",
    "name": "Auto Press X",
    "author": "POWDER-RANGER",
    "createdUtc": "2025-10-04T17:00:00Z",
    "modifiedUtc": "2025-10-04T17:00:00Z",
    "tags": ["demo", "cross"],
    "description": "Simple example macro"
  },
  "engine": {
    "fps": 60,
    "timeUnit": "ms", 
    "loop": false,
    "loopRegion": { "start": 0, "end": 2000 }
  },
  "tracks": [
    {
      "id": "track-btn-cross",
      "type": "button",
      "button": "Cross",
      "clips": [
        { "id": "c1", "start": 0, "duration": 120, "value": true },
        { "id": "c2", "start": 500, "duration": 80, "value": true }
      ]
    },
    {
      "id": "track-axis-ly",
      "type": "axis",
      "axis": "LY",
      "clips": [
        { "id": "a1", "start": 200, "duration": 300, "value": 0 },
        { "id": "a2", "start": 800, "duration": 200, "value": 128 }
      ]
    },
    {
      "id": "track-script",
      "type": "script",
      "language": "lua",
      "events": [
        { "id": "s1", "time": 1000, "callback": "onComboStart", "args": {"difficulty":"normal"} }
      ]
    }
  ],
  "conditions": [
    {
      "id": "cond-1",
      "type": "if",
      "expression": "state.health > 50 && flags.comboReady == true",
      "actions": [
        { "type": "playTrack", "target": "track-btn-cross" }
      ]
    }
  ],
  "validators": {
    "maxAnalogDeltaPerMs": 2,
    "minButtonPressMs": 30
  },
  "interop": {
    "formatVersion": 1,
    "generator": "MacroEditor 0.1-preview"
  }
}
```

JSON Schema (condensed draft):

```json
{
  "type": "object",
  "required": ["version","tracks"],
  "properties": {
    "version": {"type":"string"},
    "metadata": {"type":"object"},
    "engine": {"type":"object","properties":{"fps":{"type":"number"},"timeUnit":{"enum":["ms","frames"]}}},
    "tracks": {"type":"array","items":{"oneOf":[
      {"type":"object","required":["type","button","clips"],"properties":{"type":{"const":"button"},"button":{"type":"string"},"clips":{"type":"array"}}},
      {"type":"object","required":["type","axis","clips"],"properties":{"type":{"const":"axis"},"axis":{"type":"string"},"clips":{"type":"array"}}},
      {"type":"object","required":["type","language","events"],"properties":{"type":{"const":"script"},"language":{"enum":["lua","python"]},"events":{"type":"array"}}}
    ]}},
    "conditions": {"type":"array"}
  }
}
```

## 4) Wireframe Sketches (Markdown)

Library + Editor shell:

```
┌───────────────┬──────────────────────────────────────────────┬───────────────┐
│ Library       │ Main Canvas                                  │ Inspector     │
│ - Search 🔎   │ [Timeline Editor]                             │ [Properties]  │
│ - Tags ▣▢     │ - Tracks list + add/remove (+ Button/Axis)    │ - Selected    │
│ - Folders     │ - Ruler + playhead                            │   Clip fields │
│   • My Macros │ - Zoom slider                                 │ - Validation  │
│   • Samples   │ - Transport controls                          │   messages    │
│   • Imports   │                                              │               │
└───────────────┴──────────────────────────────────────────────┴───────────────┘
```

New Macro Wizard:

```
┌ New Macro ────────────────────────────────────────────────┐
│ Name: [______________]                                    │
│ Template: ( ) Empty  ( ) Record from controller  ( ) Sample│
│ FPS: [60]  Time unit: (•) ms  ( ) frames                  │
│ Tracks: [✓ Buttons] [✓ Axes] [✓ Script]                   │
│ [Cancel]                         [Create]                  │
└───────────────────────────────────────────────────────────┘
```

Controller Preview Overlay:

```
[DualShock glyphs] Buttons lit when active; analog bars show values (0–255)
```

## 5) Immediate TODOs for Phase 2 GUI

- Project scaffolding
  - Create WPF solution: MacroEditor.UI, MacroEditor.Core, MacroEditor.Services
  - Set up MVVM framework (CommunityToolkit.Mvvm), DI (Microsoft.Extensions.DependencyInjection)
  - Add Theming (MahApps/FluentWPF) and icon set
- Core features
  - Implement MacroDocument model and SerializationService (.ps4macro v1)
  - Implement TimelineVM with tracks, clips, selection, snapping, zooming
  - Add Transport/Playback controls wired to PlaybackEngine
  - RecordingService to capture input events from InterceptorBridge with timestamping
  - Inspector property grid with validation summaries
- Interceptor integration
  - IPC bridge wrapper for EasyHook client (start/stop, state streaming)
  - Live controller preview; latency indicator
- UX and productivity
  - Undo/Redo stack; keyboard shortcuts; context menus
  - New Macro Wizard; Import/Export (.ps4macro)
  - Sample macros folder and templates
- Quality gates
  - Unit tests for serialization and timing math
  - Basic integration test to simulate timeline playback into dummy Interceptor
  - Telemetry (opt-in) for editor performance and crashes
- Roadmap alignment
  - Script Track hooks to Lua/Python (Phase 2B)
  - Diagram assets for docs (Mermaid in docs/ and PNG exports)

---

Changelog: Initial draft for architecture, timeline, serialization, wireframes, and TODOs.
