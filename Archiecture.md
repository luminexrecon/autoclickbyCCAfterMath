# System overview

```AutoClick is a native Windows desktop app, not a browser extension or a scripting layer on top of another tool. It's written entirely in C# on .NET 8, using WPF for the UI and direct Win32 interop for everything that touches the operating system — mouse/keyboard simulation, low-level input hooks, and window management.```

The app has two halves that share one execution engine: an Auto Clicker for structured, repeatable click sequences, and a Macro Recorder for capturing and replaying real input. Both are designed around one hard requirement — the ability to target a specific background window without moving the user's real cursor or stealing focus from whatever they're actively using.

# Tech Stack 

- Runtime & UI
.NET 8 / WPF. Views are declarative XAML; a per-screen ViewModel owns state and exposes commands, keeping input logic out of code-behind wherever practical.

- Win32 interop
P/Invoke bindings for low-level mouse/keyboard hooks, SendInput/PostMessage-based input delivery, window enumeration, and DWM thumbnail capture for the window picker.

- Local persistence
Processes, macros, and settings are stored as flat local files — no bundled database engine, no telemetry service to reach for on startup.
  
- Packaging
Self-contained per-release build with an embedded application icon, custom fonts, and a fully custom window chrome (no default Windows title bar anywhere in the app).

# Automation engine

Both automation modes are built on the same idea: describe an ordered sequence of small actions, then run that sequence reliably — either once, N times, or until stopped.

## Auto Clicker — Process / Step model
A Process is a named, independently controllable job. Several can be defined and run concurrently. Each Process holds an ordered list of Steps — a mouse click or hold, a keyboard key, or a scroll action — each with its own delay. Every Process can repeat a fixed number of times or run indefinitely, and optionally apply small randomized jitter to position and timing (Humanize) so repeated runs aren't mechanically identical.

## Two input delivery modes
Steps can be delivered as real simulated input (the classic approach, requires the target to be focused) or routed directly to a specific target window's message queue — the mode that lets a Process keep clicking inside a background app while the user's mouse and keyboard stay completely free for something else.

## Macro Recorder
Records live mouse and keyboard activity through a global low-level hook, then replays it at an adjustable speed and loop count — either as real input or through the same background-window delivery path as the Auto Clicker.

## Hands-free control
- F2 Capture coordinate
- F6 Start / stop process
- F7 Stop all processes


- F8 Record macro
F9
Play macro
Registered as true global hotkeys via a low-level keyboard hook, so they work regardless of which window has focus — with input-suppression logic so normal typing in the app's own fields isn't accidentally intercepted.
