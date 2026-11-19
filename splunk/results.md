# Splunk results (CSV flow data)

Query: [`query.spl`](query.spl). It filters DDoS-labeled TCP flows to `192.168.10.50:80` and counts flows per minute. Timestamps are dataset time (about one hour ahead of Wireshark time).

- **Total DDoS-labeled flows to the server: 128,024**
- **Peak: 8,244 flows at 7/7/2017 3:57 PM**
- Baseline for comparison: 10 benign flows to the same IP and port in the morning CSV

Selected per-minute counts (dataset time):

| Minute | Flows |
|---|---|
| 3:57 | 8,244 |
| 4:04 | 6,364 |
| 4:05 | 6,273 |
| 4:06 | 5,954 |
| 4:07 | 6,053 |
| 4:08 | 6,305 |
| 4:09 | 6,466 |
| 4:10 | 6,264 |
| 4:11 | 6,300 |
| 4:12 | 6,552 |
| 4:13 | 6,403 |
| 4:14 | 6,435 |
| 4:15 | 6,265 |
| 4:16 | 1,348 |

After the 3:57 peak, the rate holds at about 6,000 to 6,500 flows per minute until the attack ends around 4:16.
