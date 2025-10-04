# Onboarding Wizard Scaffold

## 1. Step-by-Step Diagnostic/Setup Checklist

### .NET Framework Check
- [ ] Verify .NET Framework 4.7.2 or higher is installed
- [ ] Check if .NET Core 3.1+ runtime is available (if required)
- [ ] Validate assembly dependencies are resolvable
- [ ] Test for proper GAC registration of required libraries

### PS4 Remote Play Validation
- [ ] Confirm PS4 Remote Play application is installed
- [ ] Check Remote Play version compatibility (minimum version required)
- [ ] Verify Remote Play is NOT currently running (avoid conflicts)
- [ ] Locate Remote Play installation directory
- [ ] Validate RemotePlay.exe path and accessibility
- [ ] Check for required Remote Play configuration files

### Controller Detection
- [ ] Detect connected DualShock 4 controllers
- [ ] Verify controller input is readable via DirectInput/XInput
- [ ] Test controller vibration/rumble functionality
- [ ] Check for multiple controller support if applicable
- [ ] Validate button mapping configuration

### Network & Connection
- [ ] Test network connectivity to PS4
- [ ] Verify firewall rules allow Remote Play traffic
- [ ] Check port accessibility (UDP 9295, 9296, 9297)
- [ ] Validate PS4 pairing status

### Interceptor-Specific Checks
- [ ] Verify write permissions for log files
- [ ] Check for existing interceptor configuration
- [ ] Validate hook injection capabilities
- [ ] Test packet capture permissions
- [ ] Confirm no conflicting software (anti-cheat, other interceptors)

---

## 2. Draft Screens/Scripts

### Screen 1: Welcome & First-Run Setup

**Title:** Welcome to PS4 Remote Play Interceptor

**Content:**
```
Welcome!

This wizard will help you set up PS4 Remote Play Interceptor for the first time.

We'll guide you through:
✓ System compatibility checks
✓ Remote Play configuration
✓ Controller setup
✓ Ethical use guidelines

Estimated time: 3-5 minutes

[Continue] [Exit]
```

**Script Logic:**
- Display welcome message
- Set firstRun flag in config
- Initialize diagnostic results object
- Transition to System Check screen

---

### Screen 2: Ethical Use Agreement

**Title:** Ethical Use & Terms

**Content:**
```
Important: Ethical Use Guidelines

PS4 Remote Play Interceptor is designed for:
✓ Network diagnostics and analysis
✓ Accessibility modifications
✓ Personal research and development
✓ Educational purposes

This tool is NOT intended for:
✗ Cheating in online multiplayer games
✗ Violating terms of service
✗ Gaining unfair competitive advantages
✗ Harassing or disrupting other players

By continuing, you agree to use this software responsibly and in compliance
with applicable laws and service agreements.

☐ I understand and agree to the ethical use guidelines

[Back] [I Agree]
```

**Script Logic:**
- Require checkbox acknowledgment before proceeding
- Log agreement timestamp
- Store ethical agreement flag in config
- Disable [I Agree] button until checkbox is checked

---

### Screen 3: System Diagnostic

**Title:** Running System Checks

**Content:**
```
Checking your system configuration...

[✓] .NET Framework 4.7.2.................... PASSED
[✓] PS4 Remote Play Installation............ FOUND
[⚠] Remote Play Version.................... v4.5 (v5.0+ recommended)
[✓] Controller Connected................... DS4 detected
[✓] Network Connectivity................... OK
[✓] Write Permissions...................... OK
[✓] Hook Injection Support................. Available

System Status: Ready with warnings

[View Details] [Continue] [Back]
```

**Script Logic:**
- Execute each check from diagnostic checklist
- Display real-time results with status icons
- Color-code results: Green (pass), Yellow (warning), Red (fail)
- If critical failures exist, disable [Continue] and show remediation steps
- Store diagnostic results for later reference

---

### Screen 4: Remote Play Configuration

**Title:** Remote Play Setup

**Content:**
```
Configure Remote Play Integration

Remote Play Path:
[C:\Program Files (x86)\Sony\PS4 Remote Play\RemotePlay.exe] [Browse...]

Interception Mode:
( ) Standard - Hook existing Remote Play session
(•) Launch & Hook - Start Remote Play with interceptor
( ) Manual - Configure interception manually

Controller Options:
☑ Enable button remapping
☑ Enable macros
☐ Enable touchpad interception
☐ Enable motion sensor capture

Performance:
Polling Rate: [1000 Hz ▼]
Buffer Size: [64 KB ▼]

[Test Configuration] [Back] [Continue]
```

**Script Logic:**
- Auto-detect Remote Play path if possible
- Validate selected executable
- Provide browse dialog for manual selection
- Test configuration button runs quick validation
- Save configuration to config file

---

### Screen 5: Tutorial - Quick Start

**Title:** Quick Start Tutorial

**Content:**
```
How to Use PS4 Remote Play Interceptor

1. START INTERCEPTOR
   Click "Start Interceptor" button or press F9

2. LAUNCH REMOTE PLAY
   The interceptor will automatically hook into Remote Play
   (or launch it if configured)

3. MONITOR ACTIVITY
   View real-time packet capture in the main window
   Controller inputs and network traffic will be displayed

4. STOP INTERCEPTOR
   Click "Stop Interceptor" or press F10
   Remote Play will continue running normally

Keyboard Shortcuts:
• F9  - Start Interceptor
• F10 - Stop Interceptor
• F11 - Toggle Packet View
• F12 - Save Capture Log

☐ Don't show this tutorial again

[Start Tutorial Mode] [Skip to Main Application]
```

**Script Logic:**
- Offer interactive tutorial mode vs. skip option
- If tutorial mode selected, launch overlay guidance
- Store preference for hiding tutorial
- Transition to main application window

---

### Screen 6: Troubleshooting Quick Reference

**Title:** Troubleshooting & Support

**Content:**
```
Common Issues & Solutions

❌ "Failed to inject hook"
   → Run as Administrator
   → Check antivirus/firewall settings
   → Ensure Remote Play is not already running

❌ "Controller not detected"
   → Connect DS4 controller via USB or Bluetooth
   → Install DS4Windows if needed
   → Check Windows Device Manager for controller

❌ "Remote Play connection failed"
   → Verify PS4 is powered on and connected to network
   → Check firewall allows ports 9295-9297
   → Re-pair PS4 in Remote Play settings

❌ "Interceptor crashes on start"
   → Update .NET Framework
   → Check logs in %AppData%\PS4RemotePlayInterceptor\logs
   → Report issue on GitHub with log file

Need more help?
[View Full Documentation] [Check Logs] [Report Issue] [Back]
```

**Script Logic:**
- Provide quick solutions for common problems
- Link to detailed documentation
- Open log directory button
- Direct link to GitHub issues page
- Include diagnostic re-run option

---

## 3. Onboarding Sequence Flowchart

```markdown
## Onboarding Flow

```
┌─────────────────┐
│  Application    │
│  First Launch   │
└────────┬────────┘
         │
         ↓
    ┌─────────┐
    │ Check   │
    │firstRun │
    │  flag   │
    └────┬────┘
         │
    ┌────┴────┐
    │         │
   No        Yes
    │         │
    │         ↓
    │  ┌──────────────┐
    │  │   Welcome    │
    │  │   Screen     │
    │  └──────┬───────┘
    │         │
    │         ↓
    │  ┌──────────────┐
    │  │   Ethical    │
    │  │     Use      │
    │  │  Agreement   │
    │  └──────┬───────┘
    │         │
    │    ┌────┴─────┐
    │    │  Agreed? │
    │    └────┬─────┘
    │         │
    │        Yes
    │         │
    │         ↓
    │  ┌──────────────┐
    │  │   System     │◄─── Re-run if
    │  │  Diagnostic  │     fails
    │  └──────┬───────┘
    │         │
    │    ┌────┴─────┐
    │    │  Passed? │
    │    └────┬─────┘
    │         │
    │    ┌────┴────┐
    │   No        Yes
    │    │         │
    │    ↓         ↓
    │ ┌─────┐  ┌──────────────┐
    │ │Show │  │  Remote Play │
    │ │Error│  │ Configuration│
    │ │Help │  └──────┬───────┘
    │ └─────┘         │
    │                 ↓
    │          ┌──────────────┐
    │          │  Quick Start │
    │          │   Tutorial   │
    │          └──────┬───────┘
    │                 │
    │            ┌────┴─────┐
    │            │ Tutorial │
    │            │   Mode?  │
    │            └────┬─────┘
    │                 │
    │            ┌────┴────┐
    │           Yes       No
    │            │         │
    │            ↓         │
    │     ┌─────────────┐ │
    │     │  Guided     │ │
    │     │  Tutorial   │ │
    │     └──────┬──────┘ │
    │            │        │
    └────────────┴────────┘
                 │
                 ↓
          ┌─────────────┐
          │    Main     │
          │ Application │
          └─────────────┘
```
```

---

## 4. Immediate TODOs for Phase 2 Onboarding

### High Priority (Complete First)

1. **Implement Diagnostic Engine**
   - [ ] Create `DiagnosticRunner` class
   - [ ] Implement .NET framework version detection
   - [ ] Add Remote Play installation scanner
   - [ ] Build controller detection module
   - [ ] Create diagnostic result reporting structure

2. **Build Wizard UI Framework**
   - [ ] Design WPF wizard window template
   - [ ] Create navigation system (Next/Back/Cancel)
   - [ ] Implement progress indicator
   - [ ] Add screen transition animations
   - [ ] Create reusable wizard screen base class

3. **Configuration Management**
   - [ ] Design configuration schema (JSON/XML)
   - [ ] Implement config file read/write
   - [ ] Add firstRun flag handling
   - [ ] Create ethical agreement tracking
   - [ ] Build configuration validation

### Medium Priority (After Core Wizard)

4. **Individual Screen Implementation**
   - [ ] Code Welcome screen
   - [ ] Build Ethical Use screen with checkbox validation
   - [ ] Implement System Diagnostic screen with real-time updates
   - [ ] Create Remote Play Configuration screen
   - [ ] Develop Quick Start Tutorial screen

5. **Error Handling & Remediation**
   - [ ] Create detailed error messages for each diagnostic
   - [ ] Build remediation step generator
   - [ ] Add "How to Fix" links for common issues
   - [ ] Implement diagnostic re-run functionality

6. **Tutorial System**
   - [ ] Design overlay system for guided tutorial
   - [ ] Create tutorial step definitions
   - [ ] Implement highlight/pointer system for UI elements
   - [ ] Add tutorial skip/exit functionality

### Low Priority (Polish & Enhancement)

7. **Troubleshooting Integration**
   - [ ] Build troubleshooting screen
   - [ ] Add log file viewer
   - [ ] Create GitHub issue template auto-fill
   - [ ] Implement diagnostic export for support

8. **Accessibility & Localization**
   - [ ] Add keyboard navigation support
   - [ ] Implement screen reader compatibility
   - [ ] Create localization framework
   - [ ] Add language selection option

9. **Testing & Validation**
   - [ ] Write unit tests for diagnostic checks
   - [ ] Test wizard on clean Windows installations
   - [ ] Validate error handling paths
   - [ ] User acceptance testing with new users

### Technical Debt & Documentation

10. **Code Quality**
    - [ ] Document wizard architecture
    - [ ] Add XML documentation to public APIs
    - [ ] Refactor diagnostic code for maintainability
    - [ ] Create developer guide for adding new wizard screens

11. **User Documentation**
    - [ ] Write setup guide with screenshots
    - [ ] Create troubleshooting FAQ
    - [ ] Record video walkthrough
    - [ ] Build knowledge base articles

---

## Notes & Considerations

- **Performance**: Diagnostic checks should complete within 5 seconds for good UX
- **Errors**: Always provide actionable remediation steps, not just error messages
- **Flexibility**: Allow advanced users to skip wizard via command-line flag
- **Persistence**: Save wizard progress in case of crash/close
- **Telemetry**: Consider optional anonymous diagnostic reporting to improve first-run experience

---

**Next Steps:**
Begin implementation with Diagnostic Engine (TODO #1), as it's required for the System Diagnostic screen and will inform other design decisions.
