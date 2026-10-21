# Incident Investigation: Agent Tesla Infection (Unit 42 Wireshark Quiz, Jan 2023)

## Summary

On 5 January 2023 at 22:51 UTC, a Windows 11 host (DESKTOP-WIN11PC, 192.168.1.27) requested a file named Ztvfo.png over plain HTTP from 45.56.99.101. The server labelled it as an image, but the exercise background describes the traffic as an obfuscated binary. Shortly afterwards the host looked up its public IP (api.ipify.org) and sent an email over SMTP to an unrelated mailbox, containing system details and saved logins stolen from Thunderbird and Edge Chromium. The behaviour matches an Agent Tesla / OriginLogger-style information stealer, as described in the exercise background.


## Victim details

| Item | Value | Evidence |
|---|---|---|
| IP address | 192.168.1.27 | Initiates the TCP connection and the HTTP GET (packets 5 and 8) |
| MAC address | bc:ea:fa:22:74:fb (vendor lookup: Hewlett Packard) | Ethernet II source, packet 8 |
| Host name | DESKTOP-WIN11PC | SMTP EHLO (packet 647) and the stolen-data email body |
| Windows user | windows11user | Header of the stolen-data email, and its Subject line |
| OS | Microsoft Windows 11 Pro | As reported in the stolen-data email |
| CPU | Intel(R) Core(TM) i5-13600K CPU @ 5.10GHz | As reported by the malware |
| RAM | 32165.83 MB | As reported by the malware |
| Public IP | 173.66.46.112 | Mail server greeting and the email body |

## Timeline (all times UTC, 2023-01-05)

| Time | Event | Evidence |
|---|---|---|
| 22:51:00.220 | Victim looks up savory.com.bd | DNS query, packet 3 |
| 22:51:00.243 | Victim opens a TCP connection to 45.56.99.101 port 80 | SYN, packet 5 |
| 22:51:00.319 | Victim requests /sav/Ztvfo.png | HTTP GET, packet 8 |
| 22:51:01.306 | Server replies 200 OK, content type image/png | HTTP response, packet 616 |
| 22:51:25 | Victim looks up api.ipify.org (checks its own public IP) | DNS query |
| 22:51:29 | Victim looks up mail.transgear.in | DNS query |
| 22:51:30.494 | SMTP session to 204.11.58.28 port 587 starts, EHLO DESKTOP-WIN11PC | Packet 647 |
| 22:51:30.595 | Login to the mail server as marketing@transgear.in | AUTH, packet 650 |
| 22:51:31.126 | Email body with stolen data is sent | DATA, packet 662 |
| 22:53:09.156 | Session ends | QUIT, packet 686 |

## Indicators of compromise

| Type | Indicator | Context |
|---|---|---|
| Domain | savory[.]com[.]bd | First DNS lookup by the victim, at the start of the activity |
| IP | 45[.]56[.]99[.]101 | Web server the victim fetched a file from (HTTP, port 80) |
| URL path | /sav/Ztvfo.png | Requested file, served with content type image/png |
| Domain | mail[.]transgear[.]in | Looked up just before the email was sent |
| IP | 204[.]11[.]58[.]28 | SMTP server that received the stolen data (port 587) |
| Email account | marketing[@]transgear[.]in | Account used to log in to the mail server and send the data |
| Email address | zaritkt[@]arhitektondizajn[.]com | Recipient of the stolen-data email |

Notes:
- api.ipify.org is a legitimate service the malware used to check its public IP. It is not malicious by itself, so do not block it blindly.
- The sending mailbox may belong to a compromised third party. The capture alone does not show who owns it.
- Credentials seen in the capture are intentionally left out of this report.
