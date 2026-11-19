# Incident Report: HTTP GET Flood Against 192.168.10.50

**Dataset:** CICIDS2017, Friday-WorkingHours (PCAP) and matching flow CSVs
**Playbook:** CERT Société Générale IRM-4 (DDoS)
**Severity:** High
**Status:** Analysis complete. Containment and remediation are proposed, not implemented (historical data).

## 1. Summary

A public-facing Ubuntu web server (`192.168.10.50`, TCP 80) received about 95,700 `GET /` requests in roughly 20 minutes, around 81 per second. The server answered 99.97% of them but slowed from about 1 ms to over a second. The traffic appeared to come from a single address, `172.16.0.1`, which is the firewall's NAT address and also carries legitimate users.

## 2. Timeline

Splunk and Wireshark clocks differ by about one hour. Both are shown.

| Wireshark time | Dataset time | Event |
|---|---|---|
| 14:56:31 | about 3:56 PM | Attack traffic begins: repeated connect, `GET /` patterns from 172.16.0.1 |
| about 14:57 | 3:57 PM | Peak of 8,244 flows per minute in the Splunk data |
| 15:16:10 | about 4:16 PM | Last attack traffic; Splunk shows a partial final minute (1,348 flows at 4:16) |

## 3. Identification and triage

**Is it a real DDoS and not a legitimate spike?** Yes, for these reasons:

- **Concentration.** 763,838 packets came from one address (172.16.0.1) to one server, which is 99.4% of the packets sent to that server in the window.
- **Repetition.** Each connection follows the same pattern: SYN, ACK, then `GET / HTTP/1.0`, with source ports counting up one at a time (49533, 49534, 49535, and so on).
- **Rate and duration.** About 81 requests per second, sustained for about 20 minutes. The Splunk per-minute counts stay flat at roughly 6,000 to 6,500 flows after the peak.
- **Labels.** The dataset labels these flows "DDoS", and Splunk shows 128,024 of them to this server and port. A morning baseline shows only 10 benign flows to the same IP and port.
- **Attack type.** This is an application-layer attack: handshakes complete and real HTTP requests are made, so it is not a SYN flood.

**Severity: High**, because it targets a public-facing web server and its availability is the business-critical asset.

## 4. Impact analysis

| Measure | Value |
|---|---|
| Duration | about 20 minutes (14:56:31 to 15:16:10, Wireshark time) |
| Requests | 95,685 |
| Rate | about 81 requests per second |
| Requests answered | 95,661 HTTP 200 (99.97%) |
| Server response data | 1,145,960,154 bytes (about 1.15 GB), about 7.8 Mbps |
| Amplification | about 160x the size of the request packets (162x unrounded); about 24x all 48 MB sent by 172.16.0.1 |
| Server response time | about 1 ms at the start, over 1 second within seconds |

**What it means.** The server stayed up but degraded. The harm was server workload, not network bandwidth: each request was about 74 bytes on the wire but forced a full web page in reply. Real users would have seen slow page loads. The dataset has no business data, so any financial impact would be an assumption.

## 5. NAT caution

`172.16.0.1` is the firewall's NAT address. It hides the real attackers (the dataset documentation lists three upstream attack machines) and it also carries normal user traffic. Blocking it outright would cut off real users.

## 6. Recommended remediation (proposed, not implemented)

Following the IRM-4 phases:

1. **Containment.** Coordinate upstream traffic filtering or DDoS scrubbing with the network team or ISP, so malicious traffic is dropped before it reaches the NAT gateway.
2. **Mitigation.** Add connection limits and HTTP request rate limits on the web server.
3. **Recovery.** Monitor HTTP reachability and response time against the 1 ms baseline to confirm service has returned to normal.
4. **Detection.** Alert on a sudden rise in requests per second to the web server, and on response-time growth.
5. **Verification.** Review server logs and uptime monitoring to confirm what real users experienced.

**Example Nginx rate limiting (untested starting point):**

```nginx
limit_req_zone  $binary_remote_addr zone=perip:10m rate=10r/s;
limit_conn_zone $binary_remote_addr zone=conn_perip:10m;

server {
    location / {
        limit_req  zone=perip burst=20 nodelay;
        limit_conn conn_perip 20;
    }
}
```

**Caveat.** Behind NAT, `$binary_remote_addr` is the NAT address, so a per-IP limit would also throttle legitimate users sharing it. Tune limits carefully, and prefer filtering upstream or limiting on the real client address if the network can pass it.

## 7. Limits of this analysis

- No server logs or uptime data. We cannot say what real users experienced.
- Wireshark and Splunk clocks differ by about one hour. We treated them as the same attack because the per-minute counts line up.
- Flows (Splunk: 128,024) and HTTP requests (Wireshark: about 95,700) are different units, and we have not verified why the totals differ.
- The capture contains duplicate packets, so TCP retransmission counts are unreliable and were not used as evidence.
- The real attacker machines are hidden behind NAT.
- Remediation items are proposals and were not tested.

## 8. Evidence index

| Claim | Where to reproduce |
|---|---|
| 128,024 flows, 8,244 peak | [`../splunk/query.spl`](../splunk/query.spl) |
| 95,685 requests, 95,661 answered, 1.15 GB, 763,838 packets | [`../wireshark/filters.md`](../wireshark/filters.md) |
| 99.4% of traffic from 172.16.0.1 | Endpoints filter in [`../wireshark/filters.md`](../wireshark/filters.md) |
| Ticket and timeline | [`../case-management/DATA-DIG-001.md`](../case-management/DATA-DIG-001.md) |
