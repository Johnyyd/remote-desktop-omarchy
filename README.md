Setup: Sunshine + Moonlight + Tailscale on Omarchy

1 — Install Sunshine (on the Omarchy host)
Add the LizardByte repo to /etc/pacman.conf:

[lizardbyte]
SigLevel = Optional
Server = https://github.com/LizardByte/pacman-repo/releases/latest/download

sudo pacman -Sy
sudo pacman -S sunshine

2 — Permissions (required for Wayland capture + input)
sudo usermod -aG input "$USER"

sudo tee /etc/udev/rules.d/85-sunshine-input.rules >/dev/null <<'EOF'
KERNEL=="uinput", SUBSYSTEM=="misc", OPTIONS+="static_node=uinput", TAG+="uaccess", GROUP="input", MODE="0660"
EOF

sudo udevadm control --reload-rules
sudo udevadm trigger

Log out and back in so the input group takes effect. 

3 — Start Sunshine & configure
sunshine

A URL like https://localhost:47990 will print. Open it, set a username/password, and note the pairing PIN it displays. 

4 — Auto-start Sunshine on login
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/sunshine.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=Sunshine
Exec=sunshine
Terminal=false
EOF

5 — Install Tailscale on both machines
# Host (Omarchy) and client (Mac/phone/another PC)
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

Get the host's Tailscale IP:

tailscale ip -4

6 — Install Moonlight client & connect
Mac: brew install --cask moonlight (or App Store)
Steam Deck: Discover Store → Moonlight
Phone: App Store / Play Store
Linux: yay -S moonlight-qt 
In Moonlight: Add Host → enter the Tailscale IP → enter the pairing PIN from step 3 → done. 

7 — Firewall (if using ufw)
sudo ufw allow in on tailscale0 to any port 47984:47990 proto tcp
sudo ufw allow in on tailscale0 to any port 48010 proto tcp
sudo ufw allow in on tailscale0 to any port 47998:48000 proto udp

Tips

For multi-monitor, select "All Monitors" in Moonlight (not "Desktop"). 
For clipboard sync (Mac ↔ Linux): copy on Mac, then Ctrl+Alt+Shift+V inside Moonlight. 
A one-shot installer that automates steps 1–4 is at omarchy-moonlight — just ./install.sh. 
