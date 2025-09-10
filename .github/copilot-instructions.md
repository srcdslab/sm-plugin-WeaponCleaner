# Copilot Instructions for WeaponCleaner SourcePawn Plugin

## Repository Overview
This repository contains a single SourcePawn plugin called "WeaponCleaner" for SourceMod, designed to manage weapon cleanup on Source engine game servers. The plugin prevents server performance issues by limiting the number of dropped weapons and automatically removing old weapons.

**Current Version:** 2.2.2  
**Target SourceMod Version:** 1.11.0+ (minimum 1.12+ recommended)  
**Build Tool:** sourceknight

## Project Structure
```
/
├── .github/
│   ├── workflows/ci.yml          # CI/CD pipeline
│   └── dependabot.yml            # Dependency management
├── addons/sourcemod/scripting/
│   └── WeaponCleaner.sp          # Main plugin source file
├── sourceknight.yaml             # Build configuration
└── .gitignore                    # Git ignore rules
```

## Code Style & Conventions
Follow these established patterns used in this repository:

### Language Features
- Always use `#pragma semicolon 1` and `#pragma newdecls required`
- Include standard headers: `<sourcemod>`, `<sdkhooks>`, `<sdktools>`

### Naming Conventions
- **Global variables:** Prefix with `g_` (e.g., `g_hTimer`, `g_MaxWeapons`)
- **Global arrays:** Use `G_` prefix (e.g., `G_WeaponArray`)
- **Functions:** PascalCase (e.g., `OnPluginStart`, `KillWeapon`, `InsertWeapon`)
- **Local variables:** camelCase (e.g., `entref`, `client`)
- **Constants:** ALL_CAPS with underscores (e.g., `MAX_WEAPONS`, `TIMER_INTERVAL`)

### Formatting
- Use 4-space tabs for indentation
- No trailing whitespace
- Descriptive variable and function names

### Modern SourcePawn Practices
- Use `delete` instead of `CloseHandle()` - **this repo needs modernization**
- Use entity references (`EntIndexToEntRef`) for entity storage
- Prefer methodmaps over legacy Handle-based code
- All SQL operations must be asynchronous when implemented

## Plugin Architecture
This plugin implements a weapon management system with:

### Core Components
1. **Configuration System:** ConVars for max weapons (`sm_weaponcleaner_max`) and lifetime (`sm_weaponcleaner_lifetime`)
2. **Event Handling:** Round start events, client connect/disconnect
3. **SDK Hooks:** Weapon drop/equip monitoring via `SDKHook_WeaponDropPost` and `SDKHook_WeaponEquipPost`
4. **Timer System:** Periodic cleanup via `Timer_CleanupWeapons`
5. **Weapon Tracking:** Array-based storage system (`G_WeaponArray`)

### Key Functions
- `InsertWeapon()` - Adds weapons to tracking system
- `RemoveWeapon()` - Removes weapons from tracking
- `KillWeapon()` - Destroys weapons and cleans up references
- `CheckWeapons()` - Validates weapon lifetime and removes expired ones

## Build System

### sourceknight Configuration
The project uses sourceknight for building. Key configuration in `sourceknight.yaml`:
- **Dependencies:** SourceMod 1.11.0-git6934 (consider updating to 1.12+)
- **Output:** Compiled plugins go to `/addons/sourcemod/plugins`
- **Target:** WeaponCleaner plugin

### Build Commands
```bash
sourceknight build          # Build the plugin
sourceknight clean          # Clean build artifacts
```

### CI/CD Pipeline
- **Platform:** Ubuntu 24.04
- **Build Tool:** maxime1907/action-sourceknight@v1
- **Artifacts:** Automatic packaging and release creation
- **Triggers:** Push, pull request, manual dispatch

## Development Guidelines

### Making Changes
1. **Test locally** if possible with a SourceMod development environment
2. **Follow existing patterns** - this plugin has consistent code style
3. **Validate entity references** - ensure proper cleanup to prevent memory leaks
4. **Test edge cases** - weapon drops during freeze time, map changes, client disconnects
5. **Maintain backward compatibility** - this is a stable plugin used in production

### Common Patterns in This Codebase
```sourcepawn
// Entity reference usage
int entref = EntIndexToEntRef(entity);
if(!IsValidEntity(entref))
    return false;

// ConVar change handling
g_CVar_MaxWeapons.AddChangeHook(OnConVarChanged);

// SDK Hook registration
SDKHook(client, SDKHook_WeaponDropPost, OnWeaponDrop);

// Event registration
HookEvent("round_start", Event_RoundStart);
```

### Performance Considerations
- **Array management:** The plugin uses a fixed-size array (`G_WeaponArray`) for O(1) operations
- **Timer frequency:** Currently 1.0 second intervals for cleanup checks
- **Entity validation:** Always validate entities before operations
- **Memory cleanup:** Proper entity reference cleanup prevents leaks

## Testing & Validation

### Manual Testing Scenarios
1. **Weapon limits:** Drop more weapons than `sm_weaponcleaner_max` allows
2. **Lifetime expiry:** Verify weapons are removed after `sm_weaponcleaner_lifetime` seconds
3. **Round transitions:** Ensure proper cleanup on round start
4. **Client disconnect:** Verify dropped weapons from disconnecting clients are tracked
5. **Map weapons:** Confirm map-spawned weapons (with HammerID) are not cleaned

### Plugin Commands
```
sm_weaponcleaner_max <number>      // Maximum weapons (0-31)
sm_weaponcleaner_lifetime <seconds> // Weapon lifetime (0=infinite)
```

### Debugging
- Use `sm plugins list` to verify plugin is loaded
- Check `sm cvars weaponcleaner` for current settings
- Monitor server console for any error messages
- Use `sm_dump_admcache` if admin-related issues occur

## Modernization Opportunities
When making changes, consider updating legacy code:

1. **Replace CloseHandle() with delete:**
   ```sourcepawn
   // Current (legacy)
   if(g_hTimer != INVALID_HANDLE && CloseHandle(g_hTimer))
       g_hTimer = INVALID_HANDLE;
   
   // Modern approach
   delete g_hTimer;
   ```

2. **Use methodmaps for handles:**
   ```sourcepawn
   // Modern timer creation
   g_hTimer = CreateTimer(TIMER_INTERVAL, Timer_CleanupWeapons, INVALID_HANDLE, TIMER_REPEAT);
   ```

3. **Consider ArrayList for dynamic weapon tracking** (if needed for expansion)

## Common Issues & Solutions

### Build Issues
- **Missing dependencies:** Ensure SourceMod development files are available
- **Compiler errors:** Verify SourcePawn compiler version compatibility
- **Include errors:** Check include paths and SourceMod installation

### Runtime Issues
- **Entity validity:** Always validate entities before operations
- **Memory leaks:** Ensure proper cleanup in `OnPluginEnd()` and client disconnect
- **Performance:** Monitor server tick rate impact during high weapon activity

## Additional Notes
- This plugin is production-ready and stable (version 2.2.2)
- Focus on minimal, surgical changes to maintain stability
- The plugin handles edge cases well (freeze time, map weapons, client disconnects)
- Consider backward compatibility when making API changes
- Test thoroughly on development servers before production deployment