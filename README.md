# 🕵️ Kali Anonymity Toolkit
---

## ✨ Why this repo exists
A one‑stop, copy‑pasteable guide to hardening and anonymising your Kali box:  
* enable a firewall,  
* hide your IP with Tor / ProxyChains / VPN,  
* access remote Docker daemons securely,  
—all in a single Markdown scroll.

---

## 🛠️ Minimum VM specs
| Resource | Recommended |
|----------|-------------|
| **RAM**  | 6–8 GB |
| **vCPUs**| 4 |
| **Disk** | 60 GB (min) / 100 GB (ideal) |
| **Network** | NAT (safe) → *Bridge* if you need external scans |

---

## 🔥 Firewall first: choose your weapon

| Tool | TL;DR | Difficulty |
|------|-------|-----------|
| **UFW** | “Uncomplicated” wrapper for *iptables*—perfect for 80 % of labs | 🟢 Easy |
| **iptables** | Granular rules, legacy syntax | 🟠 Medium |
| **nftables** | Successor to *iptables*, modern & efficient | 🔴 Advanced |

### Quick‑start with UFW
```bash
# install & enable
sudo apt update && sudo apt install ufw -y
sudo ufw enable          # default: deny inbound, allow outbound

# allow essentials
sudo ufw allow 22/tcp     # SSH
sudo ufw allow 80,443/tcp # web
sudo ufw allow 2375/tcp   # Docker Remote API (optional, see note)

# restrict API to your box only
sudo ufw allow from <KALI_IP> to any port 2375 proto tcp

# check status
sudo ufw status verbose
```

> **Heads‑up:** Exposing port 2375 without TLS is *lab‑only*. Wrap it in SSH or TLS in production.

### iptables one‑liners (power users)
```bash
# allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
# allow Docker API from your IP
sudo iptables -A INPUT -p tcp --dport 2375 -s <KALI_IP> -j ACCEPT
# default‑deny everything else
sudo iptables -P INPUT DROP
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
# make persistent
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

---

## 🕵️‍♂️ Verify your footprint
```bash
# private IP
ip a          # or ip addr show
hostname -I   # short form

# deprecated but handy
ifconfig

# public IP
curl ifconfig.me
curl icanhazip.com
```

---

## 🛡️ ProxyChains + Tor = layered anonymity

### 1. Install ProxyChains
```bash
sudo apt update && sudo apt install proxychains4 -y
```

### 2. Edit `/etc/proxychains.conf`
```conf
dynamic_chain      # ✅ use available proxies in order
# strict_chain     # ❌ off
# random_chain     # ❌ off
proxy_dns          # anonymise DNS
# default Tor entry
socks5  127.0.0.1 9050
```

### 3. Install & run Tor
```bash
sudo apt update && sudo apt install tor -y
sudo systemctl enable --now tor
# check
tor --version
systemctl status tor
```

### 4. Test your circuit
```bash
curl --socks5 127.0.0.1:9050 https://check.torproject.org      # direct
proxychains curl ifconfig.me                                   # via ProxyChains
```

---

## 🌐 Optional: OpenVPN tunnel
```bash
sudo apt update && sudo apt install openvpn -y
sudo openvpn --config /path/to/your-provider.ovpn
curl ifconfig.me   # should show VPN IP
```

---

## 🐳 Remote Docker API (Windows host → Kali)
1. **Docker Desktop → Settings → General**  
   ✔ *Expose daemon on tcp://localhost:2375*  
2. Restart Docker.  
3. From Kali:
   ```bash
   docker -H tcp://<WINDOWS_IP>:2375 info
   ```
4. Harden with TLS or an SSH tunnel before leaving the lab.

---

## ⚠️ Troubleshooting quickies
| Problem | Fix |
|---------|-----|
| **Slow VM / crashes** | Reduce running VMs or bump host RAM/CPU |
| **No inter‑VM ping** | Check NAT vs Bridge, verify guest IP/gateway |
| **UFW locked you out** | Boot single‑user mode → `ufw disable` |

---

> **Pro tip:** snapshot before big changes—rollback beats regret.
