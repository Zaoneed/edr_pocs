# DLL Sideloading — Notepad++ / TextShaping.dll

## Technique

When Windows launches an application, it searches for required DLLs starting
from the executable's own directory before looking in system folders. DLL
Sideloading places a malicious DLL with the name of an expected (but missing)
DLL in the same directory as a legitimate signed executable. When the
application starts, it loads the malicious DLL instead, executing attacker
code inside a trusted process.

## Target

- **Application:** Notepad++ (any recent version)
- **DLL:** `TextShaping.dll` — not essential to Notepad++'s core functionality,
  so its absence or replacement does not cause a crash or visible error

The candidate DLL was identified using **Process Monitor (Procmon)**:
filtering on `notepad++.exe` with result `NAME NOT FOUND` reveals all DLLs
the application looks for but cannot find in its current directory.

## What this PoC demonstrates

When Notepad++ is launched, the malicious `TextShaping.dll` is loaded via
`DllMain` / `DLL_PROCESS_ATTACH`, which triggers execution of `calc.exe` —
proving arbitrary code execution inside a legitimate signed process.
In a real attack scenario, this would be replaced by a reverse shell or
second-stage loader.

## Files

| File | Description |
|------|-------------|
| `TextShaping.c` | Source of the malicious DLL |
| `TextShaping.dll` | Compiled malicious DLL (x64) |
| `dll_sideloading_rules.xml` | Wazuh detection rules (3 rules, levels 3/8/12) |

## Requirements

- Windows (x64), Notepad++ installed
- Write access to the Notepad++ installation directory
- `x86_64-w64-mingw32-gcc` for cross-compilation (or MSVC)

## Build

```
x86_64-w64-mingw32-gcc -shared -o TextShaping.dll TextShaping.c \
    -Wl,--subsystem,windows -lkernel32
```

## Usage

Copy `TextShaping.dll` to the Notepad++ installation directory
(`C:\Program Files\Notepad++\`), then launch Notepad++.
The calculator should open automatically, confirming execution.

## Detection

Three Wazuh rules are provided in `dll_sideloading_rules.xml`:

| Rule ID | Level | Trigger |
|---------|-------|---------|
| 100100 | 3 | Any DLL load event (Sysmon event 7) |
| 100101 | 8 | Unsigned DLL loaded into any process |
| 100102 | 12 | Unsigned DLL loaded from outside system directories |

Requires Sysmon with Image Load logging enabled (`Event ID 7`).
Be aware that these rules can generate significant false
positive volume in production environments.
