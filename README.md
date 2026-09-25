# linux-hotspot

A simple yet robust bash script to turn your Linux machine into a Wi‑Fi hotspot, sharing internet from one interface (e.g., Ethernet, USB tethering) to another (Wi‑Fi) using `nmcli` and `iptables`.

## 🚀 Features

- Create a Wi‑Fi hotspot with a custom SSID and password.
- Automatically adds the necessary `iptables` rules to share internet.
- Gracefully stops the hotspot and removes the rules.
- Dynamic connection lookup – works even if NetworkManager assigns a different profile name (e.g., `Hotspot-1`, `Hotspot-2`, …).
- Includes `start`, `stop`, `restart`, and `status` commands.
- Fully configurable – you set the interface names, SSID, and password in the script.

## 📋 Requirements

- A Linux distribution with **NetworkManager** (`nmcli` command).
- **iptables** (usually installed by default).
- A Wi‑Fi adapter that supports Access Point mode (check with `iw list | grep "Supported interface modes" -A 8`).
- An active internet connection on the interface you want to share (e.g., `eth0`, `usb0`, `enp3s0`).

## 🔧 Installation

1. **Download the script**  
   ```bash
   git clone https://github.com/eyasuasegid/linux-hotspot.git
   ```

2. **Make it executable**  
   ```bash
   cd linux-hotspot
   chmod +x hotspot
   ```

3. **(Optional) Move it to a directory in your PATH**  
   ```bash
   sudo mv hotspot /usr/local/bin/hotspot
   ```
   Now you can run `sudo hotspot start` from anywhere.

## ⚙️ Configuration

Edit the variables at the top of the script to match your setup. You can change the interface names, SSID, and password to whatever your system uses.

```bash
SSID="YourNetworkName"        # The Wi‑Fi name clients will see
PASSWORD="YourPassword"       # At least 8 characters
WLAN_IFACE="wlan0"            # Your wireless interface (e.g., wlp2s0)
ETH_IFACE="eth0"              # Your internet-connected interface (e.g., enp3s0, usb0)
```

**Note:** Interface names vary by system. Use `ip link` or `nmcli device status` to find yours. Common names:
- Wi‑Fi: `wlan0`, `wlp2s0`, `wlx...`
- Wired Ethernet: `eth0`, `enp3s0`, `enx...`
- USB tethering: `usb0`

## 📖 Usage

All commands must be run with `sudo` (or as root).

| Command          | Description                                      |
|------------------|--------------------------------------------------|
| `sudo ./hotspot start`   | Starts the hotspot and adds iptables rules.      |
| `sudo ./hotspot stop`    | Stops the hotspot and removes iptables rules.    |
| `sudo ./hotspot restart` | Restarts the hotspot (stop + start).             |
| `sudo ./hotspot status`  | Shows whether the hotspot is active on `$WLAN_IFACE`. |

### Examples

```bash
# Start the hotspot
sudo ./hotspot start

# Check status
sudo ./hotspot status
# Output: Hotspot is active on wlan0 (connection: Hotspot-3)

# Stop it
sudo ./hotspot stop
```

## 🧠 How It Works

1. **Hotspot creation**  
   The script runs `nmcli device wifi hotspot ...` – the standard NetworkManager command to create an access point. NetworkManager internally creates a connection profile (named something like `Hotspot-1`, `Hotspot-2`, …) and brings it up.

2. **Internet sharing**  
   Although NetworkManager sets the hotspot’s IPv4 method to `shared`, it sometimes fails to add the required `iptables` rules (especially when Docker or other firewall tools are present).  
   The script **explicitly adds** the necessary rules:
   - `MASQUERADE` on the internet-facing interface – allows traffic from the hotspot to be NATed to the internet.
   - `FORWARD` rules – permit traffic to flow between the two interfaces.

3. **Stopping**  
   The script finds the currently active hotspot connection on your Wi‑Fi interface using `nmcli` (dynamic lookup), brings it down, and then deletes the `iptables` rules.

## 🐛 Troubleshooting

- **Clients connect but have no internet**  
  Check that `iptables` rules were added:  
  ```bash
  sudo iptables -t nat -L POSTROUTING -v -n
  sudo iptables -L FORWARD -v -n
  ```
  If rules are missing, run `start` again or add them manually.

- **Docker conflicts**  
  Docker often modifies `iptables` and may interfere. The script’s explicit rules usually overcome this. If problems persist, try restarting Docker after stopping the hotspot.

- **Wi‑Fi adapter doesn’t support AP mode**  
  Run `iw list` and look for `AP` under "Supported interface modes". If not present, you cannot create a hotspot with that card.

- **Hotspot name is not what you set**  
  The script sets the SSID correctly. Verify with:  
  ```bash
  nmcli connection show Hotspot-* | grep 802-11-wireless.ssid
  ```

## 📄 License

This project is licensed under the MIT License – feel free to use, modify, and distribute.

---

**Enjoy sharing your internet!**
