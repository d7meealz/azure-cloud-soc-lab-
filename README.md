# Azure Cloud SOC — Purple Team Lab

A cloud-native Security Operations Center built on Microsoft Azure, combining
offensive and defensive security end to end: a live honeypot that detected real
internet attacks, detection engineering in Microsoft Sentinel, web-app penetration
testing against OWASP Juice Shop, and an Azure WAF whose logs feed back into the
same SOC.

> Built to demonstrate a full detect-and-respond loop where every attack I run
> generates telemetry the SOC can detect — the point where red team and blue team
> become one system.

## Architecture

![Architecture](diagrams/architecture.png)

Two attack surfaces feed one SOC:
- A Windows honeypot (exposed RDP) → host/identity detections
- A WAF-protected web app (OWASP Juice Shop) → web-attack detections

Both stream into a single Log Analytics workspace and Microsoft Sentinel.

## What I built

**Blue Team — Detection & Response**
- Deployed a Windows 10 honeypot with RDP exposed to the internet
- Connected it to Microsoft Sentinel via the Azure Monitor Agent (AMA) + a Data Collection Rule
- Caught real RDP brute-force attacks within the first hour (EventID 4625)
- Validated attacker IPs in VirusTotal and built a GeoIP attack map of global traffic
- Engineered tuned, MITRE ATT&CK–mapped analytics rules that auto-generated incidents
- Worked the incident queue as an analyst (triage → investigate → classify → close)

**Red Team — Offensive Testing & WAF Defense**
- Deployed OWASP Juice Shop as an Azure Container Instance
- SQL injection authentication bypass (`' OR 1=1 --`) → logged in as admin
- Cross-Site Scripting (XSS) via the search field
- Placed an Azure Application Gateway WAF (OWASP CRS) in front in Prevention mode
- Re-ran the attacks → blocked with 403 Forbidden
- Routed WAF logs into the same SOC and hunted the blocked attacks in `AGWFirewallLogs`

## Detection rules (KQL)

See the [`kql/`](kql/) folder. Highlights:
- RDP brute-force detection (failed logons grouped by source IP) — MITRE T1110.001
- Successful-logon "breach alarm" (tuned to exclude service/machine noise) — MITRE T1078
- GeoIP enrichment for the attack map
- WAF attack hunting on `AGWFirewallLogs`

## Key lessons
- Detection is about tuning, not volume — a naive `contains "success"` rule drowns
  you in false positives; filtering on EventID, LogonType, and source IP makes
  alerts meaningful.
- Context matters — distinguishing real attacker IPs from Azure platform traffic,
  and a real breach from an expected admin login, is the core of triage.
- Offense and defense are one system — the attacks generated the telemetry the SOC
  detected, closing the red/blue loop.

## Full write-up
[Presentation (PDF)](presentation/Azure_SOC_Project.pdf)

---
**Abdulrahman Alzahrani** — Penetration Tester | Cybersecurity Analyst
LinkedIn: [@d7meealz](https://linkedin.com/in/d7meealz) · Medium: [@d7meealz](https://medium.com/@d7meealz)
