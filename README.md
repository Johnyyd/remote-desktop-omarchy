# Hướng dẫn thiết lập: Sunshine + Moonlight + Tailscale trên Omarchy

## 1 - Cài đặt Sunshine (trên máy Host Omarchy)

Thêm kho lưu trữ LizardByte vào tệp `/etc/pacman.conf`:

```ini
[lizardbyte]
SigLevel = Optional
Server = https://github.com/LizardByte/pacman-repo/releases/latest/download

```

Cập nhật hệ thống và cài đặt Sunshine:

```bash
sudo pacman -Sy
sudo pacman -S sunshine

```

## 2 - Phân quyền (Yêu cầu cho Wayland capture + input)

Thêm người dùng hiện tại vào nhóm `input` và cấp quyền truy cập GPU:

```bash
sudo usermod -aG input "$USER"
sudo usermod -aG video,render "$USER"

```

Tạo quy tắc udev để cấp quyền truy cập input:

```bash
sudo tee /etc/udev/rules.d/85-sunshine-input.rules >/dev/null <<'EOF'
KERNEL=="uinput", SUBSYSTEM=="misc", OPTIONS+="static_node=uinput", TAG+="uaccess", GROUP="input", MODE="0660"
EOF

```

Tải lại quy tắc udev và kích hoạt:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger

```

> **Lưu ý:** Đăng xuất và đăng nhập lại để các nhóm quyền có hiệu lực.

## 3 - Cấu hình chống crash & Khởi chạy Sunshine

Để tránh tình trạng máy host bị đơ cứng (crash) do xung đột giữa driver card đồ họa AMD và Wayland compositor, hãy ép Sunshine sử dụng Software Encoder (tận dụng sức mạnh đa luồng của CPU Xeon):

```bash
mkdir -p ~/.config/sunshine
echo "encoder = software" >> ~/.config/sunshine/sunshine.conf

```

Khởi động Sunshine:

```bash
sunshine

```

Một URL (ví dụ: `https://localhost:47990`) sẽ xuất hiện trên terminal. Mở URL này trên trình duyệt, thiết lập **Username/Password**, và ghi chú lại mã PIN ghép nối (pairing PIN) được hiển thị.

## 4 - Tự động khởi chạy Sunshine khi đăng nhập

Tạo thư mục autostart và file cấu hình desktop:

```bash
mkdir -p ~/.config/autostart
cat > ~/.config/autostart/sunshine.desktop <<'EOF'
[Desktop Entry]
Type=Application
Name=Sunshine
Exec=sunshine
Terminal=false
EOF

```

## 5 - Cài đặt Tailscale trên cả hai máy

Thực hiện trên máy Host (Omarchy) và máy Client (Mac/điện thoại/PC khác):

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

```

Lấy địa chỉ IP Tailscale của máy Host:

```bash
tailscale ip -4

```

## 6 - Cài đặt Moonlight Client & Kết nối

Cài đặt ứng dụng Moonlight tương ứng với thiết bị Client của bạn:

* **Mac:** `brew install --cask moonlight` (hoặc tải từ App Store)
* **Steam Deck:** Discover Store → Tìm "Moonlight"
* **Điện thoại:** App Store / Play Store
* **Linux:** `yay -S moonlight-qt`

**Thiết lập trong Moonlight:**

1. Chọn **Add Host**
2. Nhập **IP Tailscale** của máy Host (đã lấy ở bước 5)
3. Nhập **mã PIN ghép nối** (từ bước 3)
4. Hoàn tất.

## 7 - Cấu hình Tường lửa (Nếu sử dụng UFW)

Mở các cổng cần thiết cho Sunshine thông qua giao diện Tailscale:

```bash
sudo ufw allow in on tailscale0 to any port 47984:47990 proto tcp
sudo ufw allow in on tailscale0 to any port 48010 proto tcp
sudo ufw allow in on tailscale0 to any port 47998:48000 proto udp

```

---

## 8 - Khắc phục lỗi Client không có âm thanh

Mặc định, Sunshine sẽ tự động ngắt âm thanh loa vật lý trên máy Host và tạo một "loa ảo" (`sink-sunshine-stereo`) để truyền sang Client. Nếu Client kết nối thành công nhưng bị mất tiếng, hãy thực hiện các bước sau trong lúc đang kết nối:

1. Chạy 2 lệnh sau trên terminal của máy Host để mở khóa và kích âm lượng cho loa ảo:

```bash
pactl set-sink-mute sink-sunshine-stereo 0
pactl set-sink-volume sink-sunshine-stereo 100%

```

2. **Khởi động lại ứng dụng phát đa phương tiện:** Các ứng dụng (như trình duyệt web, Youtube, Spotify...) đang mở từ trước có thể không tự nhận diện được loa ảo mới. Bạn cần **đóng hoàn toàn** ứng dụng đó và mở lại để luồng âm thanh được định tuyến chính xác.

---

## 💡 Mẹo hữu ích

* **Đa màn hình:** Đối với thiết lập nhiều màn hình, hãy chọn "All Monitors" trong Moonlight (không chọn "Desktop").
* **Đồng bộ Clipboard (Mac ↔ Linux):** Copy trên Mac, sau đó nhấn `Ctrl+Alt+Shift+V` bên trong cửa sổ Moonlight.
* **Cài đặt tự động:** Có sẵn một script tự động hóa tại kho lưu trữ `omarchy-moonlight` — chỉ cần chạy lệnh `./install.sh`.
