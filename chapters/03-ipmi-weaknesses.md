# IPMI Weaknesses: RAKP Hash Disclosure, Cipher 0 and Defaults

IPMI 2.0 is two decades old and still enabled on a large share of servers. Several of its weaknesses are not implementation bugs but consequences of the protocol design, so patches cannot fully fix them. This chapter explains the main historic issues, as documented by researchers around 2013, and the mitigations that still apply.

## RAKP password-hash disclosure

The RAKP handshake authenticates a user with an HMAC keyed by the user's password. The problem is the order of messages. In RAKP message 2 the BMC sends the client an HMAC-SHA1 value computed over session data (random numbers, GUID, the username, privilege level) using the stored password as the key, *before* the client has proved it knows the password. Anyone who knows or guesses a valid username can request this value and then attempt to crack it offline.

This was documented publicly in 2013 by Dan Farmer as part of a broader study of IPMI and BMC security, and Rapid7 (HD Moore and Dan Farmer) published a Metasploit module, "IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval" (`auxiliary/scanner/ipmi/ipmi_dumphashes`), disclosed 20 June 2013. The issue is tracked as CVE-2013-4786. Vendors largely responded that it was a property of the specification; Supermicro's FAQ, for example, states that the RAKP issue is an IPMI specification weakness they cannot change, while fixing cipher 0 in firmware.

Offline cracking is fast. Joshua Picolet's *Hash Crack: Password Cracking Manual* lists hashcat mode 7300, "IPMI2 RAKP HMAC-SHA1", and its benchmark table shows a rate of about 1.6 billion guesses per second for that mode on the GPU hardware it was measured on. John the Ripper also supports the format (`RAKP`). Short or dictionary passwords fall quickly.

Mitigations, because the protocol cannot be fixed:

- Disable IPMI over LAN entirely where Redfish or SSH can replace it.
- If it must stay enabled, use long random passwords for every IPMI account. Caltech's IT security guidance recommends passphrases of 20 characters or more for this reason; check your BMC's maximum IPMI password length.
- Rename or disable default usernames so attackers cannot guess the name needed to request the hash.
- Restrict UDP 623 to a management network and specific jump hosts.
- Never reuse the IPMI password elsewhere, since a cracked BMC password is often the fleet password.

## Cipher suite 0

Cipher suite 0 negotiates no authentication, no integrity and no confidentiality. On affected implementations, a client could request a session with suite 0 and any password and be logged in as any valid user, including administrators. Farmer's research, as reported publicly in 2014, estimated that a large fraction of internet-reachable IPMI 2.0 BMCs at the time accepted cipher 0. Many vendors shipped firmware that disabled it by default after 2013.

Check and disable it on your own BMCs:

```bash
# List cipher suites supported on LAN channel 1 (your own BMC)
ipmitool -I lanplus -H <bmc-ip> -U <user> -P '<pw>' channel getciphers ipmi 1
# Show per-privilege cipher suite permissions
ipmitool -I lanplus -H <bmc-ip> -U <user> -P '<pw>' lan print 1 | grep -i cipher
# Restrict: each character maps to a cipher suite ID; X = disallowed, a = admin
# (format varies by vendor; consult your BMC manual before applying)
ipmitool -I lanplus -H <bmc-ip> -U <user> -P '<pw>' lan set 1 cipher_privs XXXaXXXXXXXXXXX
```

Nmap includes an `ipmi-cipher-zero` NSE script for checking your own hosts.

## Anonymous and null users, default credentials

Other weaknesses documented around the same time:

- **Null or anonymous user:** some BMCs shipped with user ID 1 having an empty name and password, enabled for login.
- **Default credentials:** well-known factory defaults (for example `ADMIN/ADMIN` on older Supermicro boards, `root/calvin` on older Dell iDRACs) remained unchanged in many deployments. Newer hardware typically ships with unique per-device passwords printed on a label, partly due to laws such as California's SB-327 for connected devices.
- **Cleartext credential exposure:** *Nmap Network Exploration and Security Auditing Cookbook* (Calderon) uses as a teaching example some Supermicro IPMI/BMC controllers that allowed unauthenticated download of a configuration file (`/PSBlock`) containing plaintext administrative credentials, and shows how to write an NSE script to detect it. The Nmap project ships `supermicro-ipmi-conf` for that check.
- **Plaintext password storage:** because RAKP needs the raw password as an HMAC key, BMCs must store passwords in recoverable form. A BMC compromise exposes every IPMI password on it.

## What this means today

Royce Davis's *The Art of Network Penetration Testing* (Manning, 2020) still lists IPMI among the services a penetration tester checks for authentication weaknesses during internal assessments. The weaknesses are old, but they persist wherever IPMI over LAN is enabled and passwords are short or shared. A modern baseline:

| Control | Priority |
|---|---|
| IPMI over LAN disabled unless required | High |
| If enabled: only suites 3 and 17, cipher 0 disabled | High |
| Unique long random password per BMC, stored in a vault | High |
| Default and anonymous users disabled or renamed | High |
| UDP 623 reachable only from management jump hosts | High |
| Firmware at current vendor release | High |

## Worked example

*Execution status: not executed in this book's CI; verify locally.*

Audit IPMI on a BMC you own.

1. From your management workstation, check whether IPMI is reachable: `nmap -sU -p 623 --script ipmi-version <bmc-ip>`.
2. Check for cipher 0: `nmap -sU -p 623 --script ipmi-cipher-zero <bmc-ip>`. If it reports vulnerable, update firmware and restrict cipher privileges.
3. List users: `ipmitool -I lanplus -H <bmc-ip> -U <admin> -P '<pw>' user list 1`. Disable unused and anonymous entries with `user disable <id>`.
4. In a lab, with your own BMC and a test account set to a weak password, use Metasploit's `ipmi_dumphashes` module to retrieve the HMAC and confirm hashcat mode 7300 recovers it. Then set a 20-character random password and confirm it does not crack within your test budget.
5. Decide whether IPMI over LAN can be disabled entirely; if so, disable it and confirm port 623 no longer responds.

## Common pitfalls

- Believing a firmware update fixed RAKP hash disclosure; it is a protocol property.
- Disabling cipher 0 but leaving short passwords.
- Using the same IPMI password on every server.
- Leaving vendor default accounts enabled with changed passwords but guessable names.
- Running hash retrieval against devices you do not own.

## Exercises

1. Write a script that audits all your BMCs for IPMI reachability, cipher 0 and enabled default users, and outputs a CSV.
2. Estimate offline crack time for 8-, 12- and 20-character random passwords at the hashcat mode 7300 rate on your own GPU.
3. Document a migration plan from IPMI to Redfish for your monitoring and power-control tooling.

## Further reading

- Rapid7, "IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval" module: <https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_dumphashes>
- NVD, CVE-2013-4786: <https://nvd.nist.gov/vuln/detail/CVE-2013-4786>
- Caltech IMSS, "IPMI Information": <https://www.imss.caltech.edu/services/security/it-staff/ipmi-information>
- J. Picolet, *Hash Crack: Password Cracking Manual* (hashcat mode 7300)
- P. Calderon, *Nmap Network Exploration and Security Auditing Cookbook* (Supermicro IPMI recipe)
- R. Davis, *The Art of Network Penetration Testing* (Manning, 2020)
- Related books: **validator-operations-security**
