# Wireshark filters and steps

All filters were run against `Friday-WorkingHours.pcap` (SHA-256 `beff0dcce1eebc9b2454582f4dc8ed0ba0112b2c619a710bf03af93147254cd0`).

**Time window.** Wireshark's clock shows the attack from 14:56:31 to 15:16:10. The dataset lists 3:56 to 4:16 PM, so the two tools are about one hour apart. The filters below use Wireshark time with a little padding around the window.

**Tip.** If the filter bar turns red, retype the double quotes by hand. Pasting can introduce "smart quotes" that Wireshark rejects.

**How to read the numbers.** After applying a filter, open **Statistics > Capture File Properties** and read the **Displayed** column.

## 1. File properties (slide 9)

Clear the filter, open **Statistics > Capture File Properties**, click **Refresh**.

- Packets: 9,997,874
- First packet 2017-07-07 07:59:39, last packet 16:02:41, elapsed 08:03:01
- SHA-256 and SHA-1 hashes as listed in the README

## 2. Attack requests (slides 10 and 12)

```
ip.src == 172.16.0.1 && ip.dst == 192.168.10.50 && tcp.dstport == 80 && tcp.len > 0 && frame.time >= "2017-07-07 14:56:00" && frame.time <= "2017-07-07 15:17:00"
```

Result: **95,685 packets**, 7,080,690 bytes, 1,178.837 s, about **81.2 packets per second**.

## 3. Requests answered (slides 10 and 12)

```
ip.src == 192.168.10.50 && ip.dst == 172.16.0.1 && http.response.code == 200 && frame.time >= "2017-07-07 14:56:00" && frame.time <= "2017-07-07 15:17:00"
```

Result: **95,661** HTTP 200 responses (99.97% of 95,685).

## 4. Everything the server sent back (slide 12)

```
ip.src == 192.168.10.50 && ip.dst == 172.16.0.1 && frame.time >= "2017-07-07 14:56:00" && frame.time <= "2017-07-07 15:17:00"
```

Result: 516,764 packets, 1,145,960,154 bytes (about **1.15 GB**), about 7.8 Mbps.

Comparison: 1,145,960,154 / 7,080,690 is about **162x** the size of the GET request packets (the deck rounds this to about 160x). Against all 763,838 packets from 172.16.0.1 (about 48 MB), it is about 24x. Both are accurate; say which one you mean.

## 5. All attack-direction traffic (slide 11)

```
ip.src == 172.16.0.1 && ip.dst == 192.168.10.50 && frame.time >= "2017-07-07 14:56:00" && frame.time <= "2017-07-07 15:17:00"
```

Result: **763,838 packets**. Each connection shows SYN, ACK, then `GET / HTTP/1.0`, with source ports counting up (49533, 49534, 49535, ...).

## 6. Endpoints (slide 13)

Apply:

```
ip.dst == 192.168.10.50 && frame.time >= "2017-07-07 14:56:00" && frame.time <= "2017-07-07 15:20:00"
```

Then **Statistics > Endpoints**, open the **IPv4** tab, check **Limit to display filter**, and sort by **Packets** descending.

- 172.16.0.1: 763,838 packets (about 48 MB)
- 192.168.10.50 received 768,597 packets in this window
- 763,838 / 768,597 is about **99.4%** of packets to the server

## 7. I/O Graph (slide 12)

**Statistics > I/O Graphs**. Disable the default rows and add two lines (Style: Line, Interval: 1 sec):

| Name | Display filter |
|---|---|
| Requests | `ip.src == 172.16.0.1 && ip.dst == 192.168.10.50` |
| Server responses | `ip.src == 192.168.10.50 && ip.dst == 172.16.0.1` |

The x-axis is seconds since the first packet. The attack sits at roughly 25,000 to 26,200 s (14:56:31 is 25,012 s after the first packet at 07:59:39, and 15:16:10 is 26,191 s).
