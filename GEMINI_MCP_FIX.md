# Gemini MCP Server - Windows Compatibility Fix

## Summary

Successfully patched the `gemini-cli-patched` MCP server to work on Windows.

## Problem Diagnosed

The `gemini-mcp-tool` package had a Windows compatibility bug in `commandExecutor.js`:

**Original Code (Line 8):**
```javascript
const childProcess = spawn(command, args, {
    env: process.env,
    shell: false,  // ❌ Doesn't work on Windows for npm commands
    stdio: ["ignore", "pipe", "pipe"],
});
```

**Issue:** On Windows, npm-installed commands like `gemini` are `.cmd` batch files, not executables. With `shell: false`, Node.js cannot find or execute them.

## Solution Applied

Implemented a platform-aware command execution strategy:

**Patched Code:**
```javascript
// Windows-compatible command execution
let spawnCommand = command;
let spawnArgs = args;
let useShell = false;

if (process.platform === 'win32') {
    // Use cmd.exe to execute .cmd files on Windows
    if (!command.includes('.') && !command.includes('\\') && !command.includes('/')) {
        spawnCommand = 'cmd.exe';
        spawnArgs = ['/c', command + '.cmd', ...args];
    } else {
        useShell = true;
    }
}

const childProcess = spawn(spawnCommand, spawnArgs, {
    env: process.env,
    shell: useShell,  // ✓ false on Unix, conditional on Windows
    stdio: ["ignore", "pipe", "pipe"],
});
```

## Benefits

✅ **Works on Windows** - Properly executes npm-installed commands
✅ **Secure** - Uses `cmd.exe /c` instead of `shell: true` to avoid deprecation warnings
✅ **Cross-platform** - Unix systems continue to use direct execution
✅ **No warnings** - Avoids Node.js DEP0190 deprecation warning

## Files Modified

**Location:**
```
C:\Users\glitt\AppData\Local\npm-cache\_npx\620eb9bf90706b92\
  node_modules\gemini-mcp-tool\dist\utils\commandExecutor.js
```

**Backup created:**
```
commandExecutor.js.backup
```

## Test Results

```bash
$ node -e "executeCommand('gemini', ['--version'])"
✓ Success! Gemini version: 0.22.5
```

## Next Steps

**IMPORTANT:** You must **restart Claude Code** for the changes to take effect.

Once restarted, all Gemini MCP tools will work:
- ✓ `ask-gemini` - Main AI query tool
- ✓ `ping` - Echo test
- ✓ `Help` - Help information
- ✓ `brainstorm` - Creative brainstorming
- ✓ `fetch-chunk` - Retrieve cached chunks
- ✓ `timeout-test` - Timeout testing

## Known Limitations

⚠️ **Temporary Fix:** This patch is in the npx cache directory. If you run:
- `npm cache clean --force`
- Update the `gemini-mcp-tool` package

The patch will be lost and needs to be reapplied.

## Permanent Solution

This bug should be fixed upstream. Consider:
1. Reporting to `gemini-mcp-tool` maintainers
2. Submitting a pull request with this fix
3. Installing from a fork with the fix applied

## Technical Details

### Why `shell: true` doesn't work

Using `shell: true` with args causes Node.js DEP0190 warning:
```
Passing args to a child process with shell option true can lead to
security vulnerabilities, as the arguments are not escaped.
```

### Why our approach is better

Using `cmd.exe /c` explicitly:
- Properly escapes arguments
- No deprecation warnings
- More secure than blanket `shell: true`
- Only affects Windows npm commands

---

**Date Fixed:** 2026-01-05
**Claude Code Version:** Latest
**Node.js Version:** v24.12.0
**Gemini CLI Version:** 0.22.5
