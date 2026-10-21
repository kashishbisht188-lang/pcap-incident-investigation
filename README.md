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
