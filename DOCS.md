# PS4RemotePlayInterceptor - Technical Documentation

## Dependency Audit

### .NET Solution
- **Solution File**: `PS4RemotePlayInterceptor.sln`
- **Target Framework**: .NET Framework 4.0
- **IDE**: Visual Studio 2022+ (recommended for building from source)
- **Build Configuration**: Debug/Release with AnyCPU platform target

### SDK & Runtime Dependencies

#### NuGet Packages
1. **EasyHook** (v2.7.6270)
   - Primary dependency for hooking functionality
   - Location: `packages\EasyHook.2.7.6270\lib\net40\EasyHook.dll`
   - Purpose: Low-level DLL injection and API hooking
   - License: LGPL

#### System References
- System.dll
- System.Core.dll
- System.Runtime.Remoting.dll
- System.Xml.Linq.dll
- System.Data.DataSetExtensions.dll
- Microsoft.CSharp.dll
- System.Data.dll
- System.Xml.dll

#### Native DLL Dependencies
The following EasyHook native libraries are included:
- `EasyHook32.dll` / `EasyHook64.dll` - Core hooking engine (32-bit/64-bit)
- `EasyHook32Svc.exe` / `EasyHook64Svc.exe` - Service executables
- `EasyLoad32.dll` / `EasyLoad64.dll` - Loader libraries

### Installation Methods

#### Via NuGet (Recommended)
```bash
Install-Package PS4RemotePlayInterceptor
```

#### From Source
1. Clone repository
2. Open `PS4RemotePlayInterceptor.sln` in Visual Studio 2022+
3. Restore NuGet packages (EasyHook will auto-restore)
4. Build solution (F6)
5. Reference the compiled `PS4RemotePlayInterceptor.dll`

---

## Code Attribution & Architecture

### Upstream Code (Original komefai/PS4RemotePlayInterceptor)
This repository is forked from [komefai/PS4RemotePlayInterceptor](https://github.com/komefai/PS4RemotePlayInterceptor). The core interception functionality comes from the upstream project:

#### Core Components (Upstream)
- **DualShockState.cs**: Data structure representing PS4 controller state
  - Button states (Cross, Circle, Square, Triangle, L1, R1, L2, R2, L3, R3, Share, Options, PS, TouchButton)
  - Analog stick values (LX, LY, RX, RY)
  - D-pad directions
  - Touch pad coordinates
- **Hooks.cs**: Low-level hooking implementation
  - Uses EasyHook RemoteHooking API
  - Intercepts HID input reports from PS4 Remote Play client
  - Injects modified controller states back into the game stream
- **InjectionInterface.cs**: EasyHook injection entry point
  - Implements `IEntryPoint` for EasyHook
  - Manages hook lifecycle and IPC communication
- **Interceptor.cs**: High-level API for client applications
  - `Callback` delegate for receiving/modifying controller state
  - `Inject()` method to start interception
  - `EmulateController` mode for operation without physical controller
- **Watchdog.cs**: Auto-injection monitoring
  - Automatically detects PS4 Remote Play process
  - Injects hooks when Remote Play starts
  - Event handlers for injection success/failure
- **InterceptorException.cs**: Custom exception types

### Custom Extensions (POWDER-RANGER Fork)
This fork extends the original with:

#### Documentation Enhancements
- Comprehensive README with roadmap and vision
- Detailed troubleshooting section
- Use case documentation
- Community contribution guidelines

#### Planned Features (Roadmap)
- **Phase 1**: PS5 DualSense support, macOS compatibility
- **Phase 2**: Macro & scripting engine (Lua/Python bindings)
- **Phase 3**: Live screen overlay with HUD
- **Phase 4**: AI/ML integration for adaptive automation
- **Phase 5**: Plugin SDK and security hardening
- **Phase 6**: CI/CD and comprehensive testing

#### Custom Logic (Future)
- Macro recording and playback system
- Scripting API for programmable automation
- AI-powered button prediction models
- Computer vision integration for context-aware automation

> Note: Currently, most custom macro/AI features are planned (see roadmap in README). The core interception logic remains from the upstream project.

---

## Feature Comparison: Upstream vs. Fork (Macro/AI)

| Area | Upstream (komefai) | POWDER-RANGER Fork (current) | POWDER-RANGER Fork (planned) |
|---|---|---|---|
| Core interception (DS4 on PS4 RP) | Stable | Stable (minor refactors) | Continued parity
| Watchdog auto-injection | Basic | Improved event hooks | Multi-version RP client support
| Controller emulation | Limited/beta | Beta | DualSense/virtual device abstraction
| PS5 DualSense support | N/A | N/A | Full DualSense support (buttons, touch, haptics)
| macOS support | N/A | N/A | Native Mac client support
| Macro record/replay | N/A | N/A | Frame-accurate recorder and player
| Visual macro editor | N/A | N/A | GUI editor and library management
| Scripting API | N/A | N/A | Lua/Python bindings with sandboxing
| AI/ML features | N/A | N/A | Button prediction, combo assist, adaptive automation
| Overlay/HUD | N/A | N/A | Real-time input HUD and widgets
| Security | Basic | Guidance/docs | Plugin sandbox, code signing, audits
| CI/CD | Minimal | Docs WIP | Full pipelines, tests, artifacts

---

## Example Automation & Macro Scripts

### Example 1: Basic Auto-Press (Simple Macro)
Purpose: Automatically hold down X button continuously

```csharp
using PS4RemotePlayInterceptor;
using System;

namespace PS4Macros
{
  class AutoPressX
  {
    static void Main(string[] args)
    {
      Console.WriteLine("Starting Auto-Press X Macro...");
      Interceptor.Callback = new InterceptionDelegate(OnReceiveData);
      Interceptor.EmulateController = false;
      Interceptor.Watchdog.Start();
      Interceptor.Watchdog.OnInjectionSuccess = () => Console.WriteLine("✅ Injected! X button will auto-press.");
      Interceptor.Watchdog.OnInjectionFailure = () => Console.WriteLine("❌ Injection failed. Ensure PS4 Remote Play is running.");
      Console.WriteLine("Press Enter to stop...");
      Console.ReadLine();
      Interceptor.Watchdog.Stop();
    }

    private static void OnReceiveData(ref DualShockState state)
    {
      state.Cross = true; // Force X pressed
    }
  }
}
```

### Example 2: Rapid-Fire Trigger (Input Timing Macro)
Purpose: Convert R2 trigger hold into rapid-fire button taps

```csharp
using PS4RemotePlayInterceptor;
using System;
using System.Diagnostics;

namespace PS4Macros
{
  class RapidFire
  {
    private static Stopwatch timer = new Stopwatch();
    private static bool rapidFireState = false;
    private static int rapidFireIntervalMs = 100; // 10 taps/sec

    static void Main(string[] args)
    {
      Console.WriteLine("Starting Rapid-Fire R2 Macro...");
      Console.WriteLine($"Fire rate: {1000 / rapidFireIntervalMs} shots/sec\n");
      timer.Start();
      Interceptor.Callback = new InterceptionDelegate(OnReceiveData);
      Interceptor.EmulateController = false;
      Interceptor.Watchdog.Start();
      Interceptor.Watchdog.OnInjectionSuccess = () => Console.WriteLine("✅ Rapid-Fire ready! Hold R2 to activate.");
      Console.WriteLine("Press Enter to stop...");
      Console.ReadLine();
      Interceptor.Watchdog.Stop();
    }

    private static void OnReceiveData(ref DualShockState state)
    {
      if (state.R2 > 200)
      {
        if (timer.ElapsedMilliseconds >= rapidFireIntervalMs)
        {
          rapidFireState = !rapidFireState;
          timer.Restart();
        }
        state.R2 = (byte)(rapidFireState ? 255 : 0);
      }
      else
      {
        rapidFireState = false;
        timer.Restart();
      }
    }
  }
}
```

### Example 3: Custom Input Mapper (Button Remapping)
Purpose: Remap controller buttons (e.g., swap X and Circle for Japanese games)

```csharp
using PS4RemotePlayInterceptor;
using System;

namespace PS4Macros
{
  class ButtonRemapper
  {
    static void Main(string[] args)
    {
      Console.WriteLine("Starting Button Remapper...");
      Console.WriteLine("Mapping: X <-> Circle (Japanese style)\n");
      Interceptor.Callback = new InterceptionDelegate(OnReceiveData);
      Interceptor.EmulateController = false;
      Interceptor.Watchdog.Start();
      Interceptor.Watchdog.OnInjectionSuccess = () => Console.WriteLine("✅ Remapper active! X and Circle are swapped.");
      Console.WriteLine("Press Enter to stop...");
      Console.ReadLine();
      Interceptor.Watchdog.Stop();
    }

    private static void OnReceiveData(ref DualShockState state)
    {
      bool originalX = state.Cross;
      bool originalCircle = state.Circle;
      state.Cross = originalCircle;
      state.Circle = originalX;
    }
  }
}
```

### Advanced Macro Tips

#### Analog Stick Manipulation
```csharp
// Center position (neutral)
state.LX = 128; state.LY = 128;
// Full up
state.LY = 0;
// Full down
state.LY = 255;
// Full left
state.LX = 0;
// Full right
state.LX = 255;
// Diagonal up-right
state.LX = 255; state.LY = 0;
```

#### Trigger Pressure Control
```csharp
// Triggers are 0-255 pressure sensitive
state.L2 = 128; // half press
state.R2 = 255; // full press
state.R2 = 0;   // released
```

---

## Planned Scripting API: Lua and Python Templates

> Status: Planned (Phase 2). The following templates illustrate the intended scripting surface. Final APIs may differ.

### Lua Template
```lua
-- PS4RemotePlayInterceptor Lua Scripting (planned)
-- Exposed objects: pad, timing, events

function on_init()
  log.info("Lua script initialized")
  timing.set_interval_ms(5) -- callback tick interval
end

function on_frame(state)
  -- Auto-jump every 1s
  if timing.every_ms(1000) then
    state.Cross = true
  else
    state.Cross = false
  end

  -- Hold up on left stick
  state.LY = 0

  return state
end

function on_shutdown()
  log.info("Lua script stopped")
end
```

### Python Template
```python
# PS4RemotePlayInterceptor Python Scripting (planned)
# Exposed modules: pad, timing, log

from interceptor import state, timing, log

def on_init():
    log.info("Python script initialized")
    timing.set_interval_ms(5)

_last_toggle = 0
_toggle = False

def on_frame(s: state.DualShockState) -> state.DualShockState:
    global _last_toggle, _toggle
    now = timing.millis()
    if now - _last_toggle >= 100:
        _toggle = not _toggle
        _last_toggle = now
    # Rapid fire on R2 when held
    if s.R2 > 200:
        s.R2 = 255 if _toggle else 0
    return s

def on_shutdown():
    log.info("Python script stopped")
```

#### Scripting API Concepts (Planned)
- Lifecycle: `on_init()`, `on_frame(state)`, `on_shutdown()`
- Utilities: timers, schedulers, easing functions, keybinds, hot-reload
- Sandboxing: restricted modules, CPU/time budget, memory caps, no filesystem/network by default
- Packaging: `.ps4macro` bundles with metadata and permissions manifest

---

## Troubleshooting & FAQ

### Common Errors
- Error: `STATUS_INTERNAL_ERROR: Unknown error in injected C++ completion routine. (Code: 15)`
  - Solution: Restart PS4 Remote Play application and try again.
- Error: `Injection IPC failed`
  - Solution: Use compatibility mode:
    ```csharp
    Interceptor.InjectionMode = InjectionMode.Compatibility;
    Interceptor.Inject();
    ```
- Controller not responding after injection
  - Ensure Remote Play is focused and connected to console
  - Unplug/replug DualShock 4
  - Enable `EmulateController` if using without physical controller

### FAQ
- Does this work with PS5 Remote Play?
  - Planned. DualSense integration is tracked in the roadmap.
- Is macOS supported?
  - Planned. macOS client compatibility is a Phase 1 goal.
- Is this safe for online games?
  - We do not condone cheating. Use only in single-player/offline modes and respect ToS.
- Why does injection randomly fail after updates?
  - The Remote Play client changes internals across versions. Try Compatibility mode and watch for version-specific fixes.
- Can I run scripts without a controller connected?
  - Yes, via `EmulateController` mode (beta). Some games may still expect a physical device.

---

## Cross-Platform Builds & Best Practices

### Windows
- Use VS 2022, .NET Framework 4.0 target, AnyCPU
- Include EasyHook native binaries (`x86`/`x64`) alongside application
- Prefer Release builds for lower latency; enable Optimize Code

### macOS (Planned)
- Target .NET (Mono/.NET 7+) bridge for scripting layer
- Implement native injection hooks for Mac Remote Play client
- Provide notarized, signed helper binaries; respect SIP constraints

### CI/CD (Planned)
- GitHub Actions: matrix for Windows/macOS
- Artifacts: zipped binaries with correct native deps per OS/arch
- Code signing for plugins and release artifacts

### User Safety & Ethical Use
- Test in offline/single-player first; avoid competitive/online titles
- Provide visible “recording/automation active” indicators
- Add an emergency kill-switch (e.g., Esc key or hotkey)
- Log clearly what macros/scripts do; allow opt-in prompts for risky actions
- Follow community guidelines; do not bypass anti-cheat mechanisms

---

## Advanced Macro Safety Checklist
- CPU budget per frame under 2 ms; avoid heavy I/O in callbacks
- Debounce inputs; avoid oscillation that triggers anti-cheat heuristics
- Use fixed timestep where possible for deterministic playback
- Version-lock macros to Remote Play client builds if timing-sensitive

---

## Contributing
- Fork this repository
- Add macro/scripts to `examples/` with README usage
- Follow code style and add tests for new features
- Open a PR and describe your testing setup

---

Last Updated: October 4, 2025
Maintainer: POWDER-RANGER
Original Author: komefai
License: MIT
