# DATA-DIG-001: Suspected DDoS Against 192.168.10.50:80

Ticket logged in a locally hosted instance of [Catalyst](https://github.com/SecurityBrewery/catalyst) to simulate a SOC workflow. Screenshots of the dashboard and ticket are in the deck (`presentation/CYB102-capstone-deck.pdf`).

## Details

- **Incident ID:** DATA-DIG-001
- **Status:** Open, Mitigation phase
- **Severity:** High
- **Project:** CYB102 Capstone, The Data Dig
- **Dataset timeframe:** Friday, July 7, 2017 (15:56 to 16:16 dataset time / 14:56 to 15:16 Wireshark time)
- **Playbook:** CERT Société Générale IRM-4 DDoS

## Affected assets

- **Target:** 192.168.10.50 (Ubuntu web server, TCP port 80)
- **Note:** Traffic routes through the firewall/NAT gateway `172.16.0.1`. Do not blindly block this IP, because it carries legitimate user traffic and hides the true attacker IPs.

## Incident summary

At approximately 15:56 on Friday, July 7, we observed a massive volumetric HTTP GET flood targeting the public-facing web server. The attack degraded server performance, pushing response times from about 1 ms to over 1 second, though the server did not completely crash.

## Evidence and artifacts

- **Splunk flow analysis (CSV):** 128,024 flows flagged "DDoS" directed at the web server during the roughly 20-minute attack window. Peak of 8,244 flows per minute at 3:57 PM.
- **Wireshark PCAP (packet volume):** 763,838 packets from the `172.16.0.1` NAT address to the server, 99.4% of inbound traffic.
- **Wireshark PCAP (application layer):** about 95,700 HTTP `GET /` requests, averaging 81 requests per second.
- **Service degradation metrics:** 99.97% answer rate (95,661 HTTP 200 responses), returning about 1.15 GB of data, roughly 160 times the size of the inbound requests. Latency spiked within seconds of the attack starting.

## Remediation and response checklist

- [x] **Identification:** Validate attack window, target, and traffic concentration using Splunk flows and Wireshark PCAPs.
- [x] **Impact assessment:** Confirmed server degradation (high latency) but no complete outage.
- [ ] **Containment:** Coordinate upstream traffic filtering/scrubbing to drop malicious external IPs before they hit the `172.16.0.1` NAT gateway.
- [ ] **Mitigation:** Update the Nginx configuration to apply strict connection limits and HTTP request rate limits.
- [ ] **Recovery:** Monitor HTTP reachability and latency against the established 1 ms baseline to confirm service restoration.

Unchecked items are proposed actions. Because this is historical data, they were not implemented.

## Related tickets

- **Alert:** IDS Signature Match: High Volume TCP SYN/HTTP GET Flood on Port 80
- **Vulnerability:** Unrestricted HTTP Connection Limits on Web Server

## Open tasks

- Verify uptime monitoring logs to determine whether the server had a complete outage or just high latency (Angela)
- Draft the `nginx.conf` rate-limiting rules to mitigate the HTTP GET flood (Christopher)
- Upload Wireshark I/O graphs showing the 1.15 GB server response payload (Hamid)
- Upload Splunk CSV exports and timechart screenshots to the evidence locker (Frabina)

## Timeline

| Time | Event |
|---|---|
| 2017-07-07 14:56:31 (Wireshark) | Initial spike of anomalous TCP SYN packets observed hitting 192.168.10.50 |
| 2017-07-07 15:57 (dataset time) | Splunk flow logs record peak DDoS traffic at 8,244 flows per minute |
| 2017-07-07 16:16 (dataset time) | Volumetric attack traffic ceases; server response latency begins returning toward the 1 ms baseline |
