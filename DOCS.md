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

**Note**: Currently, most custom macro/AI features are planned (see roadmap in README). The core interception logic remains from the upstream project.

---

## Example Automation & Macro Scripts

### Example 1: Basic Auto-Press (Simple Macro)

**Purpose**: Automatically hold down X button continuously

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
            
            // Setup callback
            Interceptor.Callback = new InterceptionDelegate(OnReceiveData);
            Interceptor.EmulateController = false;
            
            // Start watchdog
            Interceptor.Watchdog.Start();
            Interceptor.Watchdog.OnInjectionSuccess = () => 
                Console.WriteLine("✅ Injected! X button will auto-press.");
            Interceptor.Watchdog.OnInjectionFailure = () => 
                Console.WriteLine("❌ Injection failed. Ensure PS4 Remote Play is running.");
            
            Console.WriteLine("Press Enter to stop...");
            Console.ReadLine();
            
            Interceptor.Watchdog.Stop();
        }
        
        private static void OnReceiveData(ref DualShockState state)
        {
            // Force X button pressed on every frame
            state.Cross = true;
        }
    }
}
```

**Usage**:
1. Compile and run while PS4 Remote Play is active
2. X button will be held continuously
3. Useful for: Skipping cutscenes, auto-advancing dialogue, AFK farming

---

### Example 2: Rapid-Fire Trigger (Input Timing Macro)

**Purpose**: Convert R2 trigger hold into rapid-fire button taps

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
        private static int rapidFireIntervalMs = 100; // 10 taps per second
        
        static void Main(string[] args)
        {
            Console.WriteLine("Starting Rapid-Fire R2 Macro...");
            Console.WriteLine($"Fire rate: {1000 / rapidFireIntervalMs} shots/sec\n");
            
            timer.Start();
            
            Interceptor.Callback = new InterceptionDelegate(OnReceiveData);
            Interceptor.EmulateController = false;
            Interceptor.Watchdog.Start();
            
            Interceptor.Watchdog.OnInjectionSuccess = () => 
                Console.WriteLine("✅ Rapid-Fire ready! Hold R2 to activate.");
            
            Console.WriteLine("Press Enter to stop...");
            Console.ReadLine();
            
            Interceptor.Watchdog.Stop();
        }
        
        private static void OnReceiveData(ref DualShockState state)
        {
            // Check if user is holding R2 trigger
            if (state.R2 > 200) // Threshold for "pressed" (0-255 range)
            {
                // Toggle rapid fire state every intervalMs
                if (timer.ElapsedMilliseconds >= rapidFireIntervalMs)
                {
                    rapidFireState = !rapidFireState;
                    timer.Restart();
                }
                
                // Apply rapid fire
                state.R2 = (byte)(rapidFireState ? 255 : 0);
            }
            else
            {
                // Reset when R2 released
                rapidFireState = false;
                timer.Restart();
            }
        }
    }
}
```

**Usage**:
1. Run macro and start PS4 Remote Play
2. Hold R2 in-game
3. Trigger will pulse on/off at configured interval
4. Useful for: Rapid-fire weapons, charge-cancel exploits, rhythm games

**Configuration**:
- Adjust `rapidFireIntervalMs` for different fire rates
- Lower = faster (e.g., 50ms = 20 shots/sec)
- Higher = slower (e.g., 200ms = 5 shots/sec)

---

### Example 3: Custom Input Mapper (Button Remapping)

**Purpose**: Remap controller buttons (e.g., swap X and Circle for Japanese games)

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
            
            Interceptor.Watchdog.OnInjectionSuccess = () => 
                Console.WriteLine("✅ Remapper active! X and Circle are swapped.");
            
            Console.WriteLine("Press Enter to stop...");
            Console.ReadLine();
            
            Interceptor.Watchdog.Stop();
        }
        
        private static void OnReceiveData(ref DualShockState state)
        {
            // Save original button states
            bool originalX = state.Cross;
            bool originalCircle = state.Circle;
            
            // Swap the buttons
            state.Cross = originalCircle;
            state.Circle = originalX;
            
            // Additional mappings (examples):
            // Swap Square and Triangle
            // bool originalSquare = state.Square;
            // bool originalTriangle = state.Triangle;
            // state.Square = originalTriangle;
            // state.Triangle = originalSquare;
            
            // Remap L3 to L1 (for games with awkward sprint controls)
            // state.L1 = state.L3;
        }
    }
}
```

**Usage**:
1. Modify button swap logic as needed
2. Run macro before starting game
3. Buttons will be remapped according to your configuration
4. Useful for: Accessibility, Japanese game imports, personal preferences

**Common Remapping Scenarios**:
- Japanese games: X ↔ Circle
- Accessibility: Map hard-to-reach buttons to easier ones
- FPS comfort: Swap R3/L3 with bumpers for crouch/sprint
- Fighting games: Custom button layouts for combos

---

## Advanced Macro Tips

### Analog Stick Manipulation
```csharp
// Center position (neutral)
state.LX = 128;
state.LY = 128;

// Full up
state.LY = 0;

// Full down  
state.LY = 255;

// Full left
state.LX = 0;

// Full right
state.LX = 255;

// Diagonal movement (up-right)
state.LX = 255;
state.LY = 0;
```

### Trigger Pressure Control
```csharp
// Triggers are 0-255 pressure sensitive
state.L2 = 128;  // Half press
state.R2 = 255;  // Full press
state.R2 = 0;    // Released
```

### Safety Considerations
1. **Always test macros in offline/single-player modes first**
2. **Respect game Terms of Service** - Many online games prohibit automation
3. **Use responsibly** - Macros can provide unfair advantages in competitive play
4. **Monitor CPU usage** - Excessive callback processing can cause lag
5. **Include kill-switch** - Always provide a way to disable the macro quickly

---

## Future Enhancements

Planned features for macro system:
- **Macro recording**: Record input sequences and replay them
- **Scripting engine**: Lua/Python support for complex logic
- **Conditional branching**: If/else logic based on game state
- **Computer vision integration**: React to on-screen events
- **Timing controls**: Frame-perfect input for speedruns
- **Macro library**: Share community-created automation scripts

See roadmap in main README for detailed timeline.

---

## Contributing

Want to add your own macro examples or improve this documentation?
1. Fork this repository
2. Add your macro scripts to a new folder (e.g., `examples/macros/`)
3. Document usage clearly
4. Submit a pull request

We welcome contributions from the community!

---

**Last Updated**: October 4, 2025  
**Maintainer**: POWDER-RANGER  
**Original Author**: komefai  
**License**: MIT
