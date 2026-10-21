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


## How I found it

| # | Question | Answer | How I found it |
|---|---|---|---|
| 1 | Start of malicious traffic | 2023-01-05 22:51 UTC | Filter `http`: first request is GET /sav/Ztvfo.png, packet 8, 22:51:00.319 UTC |
| 2 | Victim IP | 192.168.1.27 | Same GET: this host opens the TCP connection (packet 5) and sends the request, so it is the client |
| 3 | MAC address | bc:ea:fa:22:74:fb | Packet 8, Ethernet II, Source field |
| 4 | Host name | DESKTOP-WIN11PC | Filter `smtp.req`: EHLO line, packet 647 |
| 5 | Windows user | windows11user | Follow TCP Stream on the SMTP session: `User Name:` line in the email header block |
| 6 | RAM | 32165.83 MB | Same stream: `RAM:` line |
| 7 | CPU | Intel(R) Core(TM) i5-13600K CPU @ 5.10GHz | Same stream: `CPU:` line |
| 8 | Public IP | 173.66.46.112 | Same stream: `IP Address:` line, and the mail server's greeting |
| 9 | Stolen login data | Email-client logins (Thunderbird) and saved website logins (Edge Chromium) | Same stream: `Application:` and `URL:` fields |

**Method in short:** I set the time to UTC, filtered on the victim, listed DNS lookups to spot the odd domains, checked what the host fetched over HTTP, then found the SMTP session and read the email body with Follow TCP Stream. Host name and public IP each appeared in two places, which confirmed them.

## Recommendations

| Action | Why |
|---|---|
| Isolate the host (DESKTOP-WIN11PC, 192.168.1.27) from the network | Stops further data theft and stops it spreading, while it is investigated |
| Reset every password that was stored in Thunderbird and Edge on this PC, and review those accounts for suspicious logins | The attacker already has these logins and can use them immediately |
| Block 45[.]56[.]99[.]101 and 204[.]11[.]58[.]28, and review traffic to savory[.]com[.]bd and mail[.]transgear[.]in | Cuts the payload download and the data-exfiltration paths |
| Report the sending mailbox (marketing[@]transgear[.]in) to its provider | Shuts down the channel the stolen data was sent through |
| Clean or reimage the PC, and find out how the file got there | Removes the malware and closes the initial entry route |
| Alert on workstations that send unencrypted SMTP to outside mail servers | Would catch this behaviour early in future |
| Do not block api[.]ipify[.]org on its own | It is a legitimate service, and blocking it causes false positives |

## Source

The packet capture and the quiz questions are from the Unit 42 Wireshark Quiz, January 2023 (Palo Alto Networks): https://unit42.paloaltonetworks.com/january-wireshark-quiz/

The capture is not included in this repository, because it contains a malware payload. The analysis and write-up are my own.
