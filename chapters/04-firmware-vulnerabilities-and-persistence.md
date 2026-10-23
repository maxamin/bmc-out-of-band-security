# BMC Firmware Vulnerability Classes, Persistence and Supply Chain

Beyond IPMI's protocol flaws, BMC firmware has a long record of ordinary software vulnerabilities with extraordinary consequences. This chapter groups publicly reported issues into classes, explains how attackers have used BMCs for persistence, and covers supply-chain risks. Each case below is described as reported by the researchers or vendors named; check the original advisories for affected versions.

## Vulnerability classes

| Class | Typical cause | Public example |
|---|---|---|
| Web/API authentication bypass | Flawed checks in C/CGI or Lua handlers | HPE iLO 4 CVE-2017-12542; AMI MegaRAC CVE-2024-54085 |
| Memory corruption in network services | C parsers for HTTP, IPMI, KVM, virtual media | Many vendor advisories; iLO 4 research below |
| Host-to-BMC privilege boundary | Hardware bridges or local interfaces trusted by default | Pantsdown (CVE-2019-6260) |
| Weak or missing update authentication | Unsigned images, downgrade allowed | Noted in *Rootkits and Bootkits*; iLOBleed downgrade concerns |
| Credential handling | Plaintext storage, hard-coded accounts | IPMI password storage (Chapter 3) |
| Virtual media / KVM exposure | Weak authentication on USB-over-IP services | Eclypsium's 2019 "USBAnywhere" report on Supermicro BMCs |

## Selected public research

**HPE iLO 4 (2017–2018).** Airbus security researchers Fabien Périgaud, Alexandre Gazet and Joffrey Czarny reported CVE-2017-12542, an authentication bypass and remote code execution in the iLO 4 web server, fixed in iLO 4 firmware 2.54. Their SSTIC 2018 paper "Subverting your server through its BMC: the HPE iLO4 case" described how they reverse-engineered the firmware, and showed how control of iLO could be used to compromise the host. *Rootkits and Bootkits* recommends the paper as a detailed case study, and Sean Metcalf's DEF CON 26 talk "Exploiting Administrator Insecurities" cited it as a reason to treat BMCs as tier-0 assets in Active Directory environments, noting coverage that the bypass could be triggered with a long string of "A" characters in an HTTP header.

**Pantsdown (2019).** Stewart Smith of IBM disclosed CVE-2019-6260: ASPEED AST2400 and AST2500 BMCs implement AHB bridges that, unless explicitly disabled by firmware, let software on the host read and write the BMC's physical address space over PCIe or LPC. A root user on the host could therefore modify BMC firmware. The issue affected multiple firmware stacks, including OpenBMC, Supermicro and AMI-based firmware, and Eclypsium later reported that some vendors' servers remained affected years afterwards. The lesson is that the host-to-BMC boundary is a security boundary that firmware must configure correctly.

**iLOBleed (2021).** Amnpardaz reported malware hidden in HPE iLO firmware, observed in incidents since 2020 according to its report. As described in that report and press coverage, the implant survived OS reinstallation, displayed fake firmware-upgrade progress in the web interface while blocking real updates, and was used to wipe servers' disks. The case is often cited as the first publicly documented in-the-wild BMC firmware implant; treat attribution claims in secondary coverage with caution.

**AMI MegaRAC (2022–2025).** Eclypsium published several sets of vulnerabilities in AMI's MegaRAC SPx firmware, which many server vendors license. The most severe, CVE-2024-54085, allows authentication bypass through the Redfish Host Interface by manipulating headers so a request appears to come from the host. AMI released patches in March 2025; CISA added the CVE to the Known Exploited Vulnerabilities catalogue on 25 June 2025. Because one firmware stack sits under many brands, downstream patch availability lagged according to reporting.

## Persistence below the OS

A compromised BMC gives an attacker options the host cannot see:

- **Firmware implant on the BMC** that survives OS reinstalls and disk replacement.
- **Host firmware modification**: many BMCs can reflash the host BIOS/UEFI, enabling bootkits.
- **Virtual media boot** of an attacker-controlled image, or console access to recovery modes.
- **Covert channel**: management traffic on a separate NIC or sideband, invisible to host monitoring; PLATINUM's use of AMT Serial-over-LAN, reported by Microsoft in 2017, shows the idea.
- **Fake updates** that report success without changing firmware, as described in the iLOBleed report.

*Rootkits and Bootkits* emphasises that a BMC update process that does not authenticate network-delivered images "is just as vulnerable as any security-through-obscurity software installation scheme", and Alexander Ermolov's ZeroNights 2021 talk "Data-Only Attacks Against UEFI BIOS" notes the growing UEFI runtime surface exposed to Redfish and similar services. Recovery therefore cannot rely on the BMC's own report of its version.

## Supply chain

- **Shared code**: one vendor stack (AMI, ASPEED SDKs, OpenBMC) appears under many brands, so one bug spreads widely and patches depend on each OEM.
- **Old components**: BMC firmware often bundles outdated open-source libraries (web servers, OpenSSL, BusyBox). Firmware analysis tools can list them.
- **Refurbished and second-hand servers**: previous owners' BMC configuration, accounts and possibly firmware remain unless reset and reflashed.
- **Hardware roots of trust**: modern platforms add hardware roots of trust (for example OpenTitan-style or vendor silicon RoTs) that verify BMC and host firmware before boot. Check whether your platform has one and whether it is enabled.

## Worked example

*Execution status: not executed in this book's CI; verify locally.*

Analyse a vendor BMC firmware image you have downloaded legitimately, without touching any live device.

1. Download the current and one older firmware image for a BMC model you own from the vendor's support site and verify published checksums.
2. Extract with `binwalk -e <image>` or `unblob <image>` in an empty working directory.
3. Locate the root filesystem and list third-party components and versions, for example `strings` on `lighttpd`, `libssl.so`, `busybox`, or use a firmware SBOM tool.
4. Compare the two versions: which components changed, and does the vendor's advisory list match?
5. Check whether the image carries a signature block and whether the vendor documents signature enforcement and anti-rollback for updates.

## Common pitfalls

- Trusting the BMC's web UI for its own firmware version after a suspected compromise.
- Assuming an OEM has shipped a fix because the upstream firmware vendor has.
- Redeploying refurbished servers without a BMC factory reset and reflash.
- Leaving host-to-BMC interfaces unrestricted on multi-tenant bare-metal hosts.
- Publishing details of a new BMC bug before the vendor has a fix.

## Exercises

1. For each server model you run, list the firmware stack lineage (OEM, AMI/OpenBMC/other) and the last three security advisories.
2. Write a recovery procedure for a suspected BMC compromise that does not rely on the BMC itself, using vendor out-of-band reflash or hardware recovery where available.
3. Build an SBOM for one BMC firmware image and identify the oldest bundled component.

## Further reading

- F. Périgaud, A. Gazet, J. Czarny, "Subverting your server through its BMC: the HPE iLO4 case", SSTIC 2018: <https://airbus-seclab.github.io/ilo/>
- S. Metcalf, "Exploiting Administrator Insecurities", DEF CON 26 (2018)
- A. Matrosov, E. Rodionov, S. Bratus, *Rootkits and Bootkits* (2019), "BMC Chip Vulnerabilities"
- A. Ermolov, "Data-Only Attacks Against UEFI BIOS", ZeroNights 2021
- Pantsdown, CVE-2019-6260: <https://nvd.nist.gov/vuln/detail/cve-2019-6260>
- Censys advisory on CVE-2024-54085: <https://censys.com/advisory/cve-2024-54085/>
- Related books: **firmware-fuzzing-llm-peripherals**, **web3-supply-chain-security**
