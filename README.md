# BMC and Out-of-Band Management Security

> IPMI, Redfish and BMC firmware: the management plane beneath the OS, from exposure to persistence.

## Contents

<!-- toc:start -->
1. [The Management Plane Beneath the OS: Problem and Threat Model](chapters/01-problem-and-threat-model.md)
2. [BMC Architecture, IPMI and Redfish](chapters/02-bmc-architecture-ipmi-redfish.md)
3. [IPMI Weaknesses: RAKP Hash Disclosure, Cipher 0 and Defaults](chapters/03-ipmi-weaknesses.md)
4. [BMC Firmware Vulnerability Classes, Persistence and Supply Chain](chapters/04-firmware-vulnerabilities-and-persistence.md)
5. [Network Isolation, Hardening and Monitoring](chapters/05-isolation-hardening-and-monitoring.md)
6. [A BMC Security Programme, Node Hosting and Research Directions](chapters/06-programme-checklist-and-node-hosting.md)
<!-- toc:end -->

## How to read this

Read chapters in order, or jump to what you need. Every chapter ends with exercises and further reading. See [RESOURCES.md](RESOURCES.md) and [GLOSSARY.md](GLOSSARY.md).

## Build

```bash
mdbook serve   # optional: renders SUMMARY.md as a website
```

## License

Text: CC BY-SA 4.0. Code snippets: MIT. See [LICENSE](LICENSE).
