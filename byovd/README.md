# BYOVD — Bring Your Own Vulnerable Driver

## Technique

BYOVD abuses a legitimate but vulnerable signed driver to gain arbitrary
kernel read/write primitives. Since modern Windows requires drivers to be
signed, loading a custom malicious driver is not straightforward — BYOVD
works around this by loading an already-signed driver that contains a known
vulnerability instead.

The driver used here is **dbutil_2_3.sys** (Dell), which exposes vulnerable
IOCTL handlers allowing arbitrary kernel memory access (CVE-2021-21551).

## What this PoC demonstrates

The PoC performs a **token stealing** attack:
1. Locate the EPROCESS structure of the System process (PID 4) in kernel memory
2. Read its SYSTEM token
3. Walk the EPROCESS linked list to find the current process
4. Overwrite the current process token with the SYSTEM token

This results in full SYSTEM privileges from a user-mode process with local
admin rights.

## Credits

This PoC is heavily inspired by
[mathisvickie/CVE-2021-21551](https://github.com/mathisvickie/CVE-2021-21551).
The main adaptation was updating the EPROCESS field offsets for Windows 10 22H2:

```c
// Original (Windows 10 2004 / 20H2)
DWORD EPROCESS_ActiveProcessLinks = 0x2e8;
DWORD EPROCESS_Token              = 0x348;

// Adapted for Windows 10 22H2
DWORD EPROCESS_ActiveProcessLinks = 0x448;
DWORD EPROCESS_Token              = 0x4b8;
```

## Files

| File | Description |
|------|-------------|
| `CVE-2021-21551.c` | Token stealing PoC (adapted from mathisvickie) |
| `dbutil_2_3.c` | Helper code for IOCTL communication with the driver |
| `dbutil_2_3.sys` | Vulnerable Dell driver (do not load outside a lab) |
| `byovd_dbutil_2_3_rule.xml` | Wazuh detection rule (hash-based, level 15) |

## Requirements

- Windows 10 22H2 (x64)
- Local administrator rights
- HVCI disabled (or dbutil_2_3.sys not on the block list)
- Visual Studio Build Tools (for compilation)

## Build

Open an **x64 Native Tools Command Prompt for VS 2022** as administrator:

```
cl.exe /TP CVE-2021-21551.c /link ntdll.lib
```

## Usage

```
sc create DBUtil_2_3 type=kernel start=demand binPath="<path>\dbutil_2_3.sys"
sc start DBUtil_2_3
CVE-2021-21551.exe
whoami   # should return: nt authority\system
```

## Detection

The Wazuh rule in `byovd_dbutil_2_3_rule.xml` triggers a **level 15 critical
alert** on any Sysmon Driver Load event (event ID 6) matching the SHA256 hash
of the known vulnerable version of dbutil_2_3.sys.

Note: Once active, it can suppress Sysmon kernel callbacks and blind Wazuh entirely.
