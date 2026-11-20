# Home Network Security Assessment

A hands-on assessment of my own home Wi-Fi network, done in November 2026 as practice after learning Linux and network basics on OverTheWire Bandit.

> **Scope and ethics:** this was my own network and my own devices. I only *observed* (discovery, port scans, read-only queries and one harmless test command). No exploitation or password guessing. Identifying details (MAC addresses, serial numbers, device IDs, network name) have been removed.

## Tools used

| Tool | Purpose |
|---|---|
| `ipconfig`, `route` | Find my own address and the router's address |
| `arp -a` | List devices my machine had recently talked to |
| `nmap` | Discover devices and open ports, identify services |
| `curl` | Send web requests to a device's control service |

## 1. Finding my place on the network

```bash
ipconfig getifaddr en0                  # my address on the network
route -n get default | grep gateway     # the router's address
```

The network uses a standard private range (`192.168.x.0/24`), with the router at `.1`.

## 2. Discovering devices: one tool's view is never the full picture

| Method | Devices found | Why |
|---|---|---|
| `arp -a` | 12 | Only devices my Mac had recently talked to |
| `nmap -sn` (no admin rights) | 5 | Can only "knock" with web-style connections; sleeping phones, firewalls and IoT devices often ignore them |
| `sudo nmap -sn` (admin) | **19** | Uses ARP, which every device on the local network must answer |

**Lesson:** a device that doesn't show up in one scan isn't necessarily absent. Combining methods gives a more complete picture.

### What the 19 devices turned out to be

nmap looks up the manufacturer from the first half of each hardware (MAC) address. This is often the **chip maker, not the brand on the box**, so these are educated guesses:

| Category | Count | Notes |
|---|---|---|
| Mesh router units | 3 | Matching hardware addresses, so one router system |
| Smart TVs / streaming devices | 4 | Including a Roku, and TVs with casting features |
| Game console | 1 | Nintendo |
| Laptops | 2 | Identified by Wi-Fi chip makers |
| IoT devices | 2 | Identical chips commonly used in smart plugs/cameras |
| Phones/tablets/Macs | 7 | Used **randomised "private" MAC addresses**, so the maker was unknown |

Randomised MAC addresses (second character `2`, `6`, `a` or `e`) are a privacy feature of modern phones and computers. They make devices harder to track, and harder to identify in a scan.

## 3. Router

```bash
sudo nmap 192.168.x.1
```

| Port | Service | Assessment |
|---|---|---|
| 53 | DNS | Normal, needed |
| 80 / 443 | Admin web page | Normal on the inside of the network |
| 1900 | UPnP | Review: lets devices open ports to the internet automatically, without asking |

**Finding:** normal exposure. UPnP is a convenience feature that a compromised IoT device could misuse; many hardening guides suggest disabling it unless something needs it.

## 4. IoT devices

```bash
sudo nmap -sV <device1> <device2>
```

Both devices had **999 of 1,000 common ports closed** and a single open port reported as `tcpwrapped`: the device accepts a connection and immediately closes it without identifying itself.

**Finding:** small attack surface on the local network. No telnet, FTP, open admin pages or unprotected video streams.

**Limits of this result:** the scan can't see the devices' **cloud side** (account security, the vendor's servers), UDP ports, or software flaws behind the one open port. For cameras especially, the cloud account is usually the bigger risk.

## 5. Smart TVs and streaming devices

| Device | Result | Assessment |
|---|---|---|
| Phone | All 1,000 ports **filtered** (no response at all) | Firewalled, very locked down |
| TV 1 | AirPlay receiver + one unidentified service | Casting feature working as designed |
| TV 2 | 9 open ports: Google Cast, AirPlay, encrypted web services, UPnP/DLNA media sharing | Largest attack surface on the network, typical for a smart TV |
| Roku | All 1,000 default ports closed | Looked silent, but see below |

Port states:
- **open:** something answers
- **closed:** the device refuses the connection
- **filtered:** no answer at all, usually a firewall

## 6. Finding what a default scan missed: the Roku

nmap's default scan checks only the 1,000 most common ports out of 65,535. Roku devices are known to use **port 8060** for their remote-control service (Roku's External Control Protocol), which isn't in that list.

```bash
sudo nmap -sV -p 8060 <roku>
```

Port 8060 was **open**. A clean default scan does not mean a device has no open ports.

### Testing what the service allows

| Request | Response | Meaning |
|---|---|---|
| `curl http://<roku>:8060/query/device-info` | Full device details, **no password** | Information disclosure (see below) |
| `curl http://<roku>:8060/query/apps` | `ECP command not allowed in Limited mode.` | Installed apps hidden |
| `curl -X POST http://<roku>:8060/keypress/Home` | *(no output)* | Unclear… |
| `curl -i -X POST http://<roku>:8060/keypress/Home` | `HTTP/1.1 403 Forbidden` | Remote control **blocked** |

The device info that anyone on the Wi-Fi could read included: model, exact software version, serial number, device and advertising IDs, Wi-Fi network name, language, locale and time zone. It also confirmed the protective settings: control mode `limited`, developer mode off.

**Lesson learned the hard way:** the button-press request returned *nothing*, which looked like it might have succeeded. Only checking the HTTP status code (`curl -i`) showed it was actually refused. **Silence is not success; check the evidence.**

**Lesson 2:** even while refusing, the device announced its software version in a response header (`Server: Roku/16.x`). These **banners** help attackers look up known flaws, which is one reason keeping devices updated matters.

## Summary

| Device | Exposure |
|---|---|
| Router | 🟢 Normal (consider disabling UPnP) |
| Phone | 🟢 Fully firewalled |
| IoT devices (2) | 🟢 One quiet port each on the local network |
| Roku | 🟡 Basic device info readable; apps and control blocked |
| TV 1 | 🟡 AirPlay + one unknown service |
| TV 2 | 🟡 Many casting services: largest attack surface |

Nothing alarming was found. Open ports mostly matched features working as designed.

## Recommendations

1. Keep router firmware and smart TV/Roku software updated
2. Disable UPnP on the router unless something needs it
3. Put IoT devices and guests on a separate guest network
4. Turn off casting features that aren't used, or require a code to connect
5. Use strong, unique passwords and two-factor login on IoT app accounts
6. Confirm WPA2/WPA3 Wi-Fi encryption, a long Wi-Fi password, and WPS off

## What this assessment did *not* cover

- All 65,535 TCP ports on every device, and UDP ports
- Checking discovered software versions against known vulnerabilities
- Wi-Fi configuration and router firmware version
- How the network looks from the internet side
- The devices' cloud services

"Secure" is never a final state: a new flaw can be published tomorrow. The goal is to **reduce risk**, not to declare a network safe.

## Key takeaways

1. **Combine tools.** ARP found 19 devices where a basic ping scan found 5.
2. **Know what default scans skip.** A targeted check found a port the default scan missed.
3. **Find → identify → test → explain.** The basic loop of a security assessment.
4. **Check evidence, don't assume.** Status codes, not silence.
5. **More features = more open doors.** The smart TV with the most features had the most exposure.
6. **Only test what you own or have written permission to test.**
