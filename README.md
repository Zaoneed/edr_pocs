# EDR Bypass Research — Wazuh Evaluation

This repository contains two proof-of-concept implementations developed
during a research internship at Télécom Paris (INFRES department), as part
of a study evaluating Wazuh's ability to detect common EDR bypass techniques.

## Structure

```
byovd/              — BYOVD via dbutil_2_3.sys (CVE-2021-21551)
dll_sideloading/    — DLL Sideloading via Notepad++ / TextShaping.dll
```

## Context

Both PoCs were built to test the detection capabilities of Wazuh (open source
EDR platform) against real-world evasion techniques. Each folder contains
the attack code, the vulnerable/malicious component, and the corresponding
Wazuh detection rule(s).

## Disclaimer

This project is strictly for educational and research purposes.
All experiments were conducted in an isolated lab environment.
Do not use any of this on systems you do not own or have explicit
written permission to test.
