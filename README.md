# pcap-incident-investigation
# Incident Investigation: Agent Tesla Infection (Unit 42 Wireshark Quiz, Jan 2023)

## Summary

On 5 January 2023 at 22:51 UTC, a Windows 11 host (DESKTOP-WIN11PC, 192.168.1.27) requested a file named Ztvfo.png over plain HTTP from 45.56.99.101. The server labelled it as an image, but the exercise background describes the traffic as an obfuscated binary. Shortly afterwards the host looked up its public IP (api.ipify.org) and sent an email over SMTP to an unrelated mailbox, containing system details and saved logins stolen from Thunderbird and Edge Chromium. The behaviour matches an Agent Tesla / OriginLogger-style information stealer, as described in the exercise background.
