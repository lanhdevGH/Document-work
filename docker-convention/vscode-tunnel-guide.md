# Hướng dẫn cài đặt và sử dụng VS Code Tunnel

Visual Studio Code Tunnel cho phép bạn kết nối từ xa vào server và làm việc trên môi trường phát triển từ bất kỳ đâu. Dưới đây là các bước chi tiết để thiết lập và sử dụng VS Code Tunnel.

## Bước 1: Cài đặt Visual Studio Code

- Đầu tiên, bạn cần cài đặt VS Code trên máy tính cá nhân của mình (nếu chưa cài)
- Tải VS Code từ trang chính thức: https://code.visualstudio.com/

## Bước 2: Cài đặt Visual Studio Code trên server

- Kết nối đến server của bạn thông qua SSH hoặc phương thức truy cập từ xa khác
- Cài đặt VS Code trên server (ví dụ cho Debian/Ubuntu):

```bash
sudo apt update
sudo apt install wget gpg apt-transport-https
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
rm -f packages.microsoft.gpg
sudo apt update
sudo apt install code
```

## Bước 3: Cài đặt Visual Studio Code CLI trên server

- Nếu server không có giao diện đồ họa, bạn có thể cài đặt chỉ riêng VS Code CLI:

```bash
curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz
tar -xf vscode_cli.tar.gz
```

## Bước 4: Tạo và khởi động VS Code Tunnel trên server

- Sau khi cài đặt VS Code CLI trên server, thực hiện lệnh sau để tạo tunnel:

```bash
./code tunnel
```

- Hệ thống sẽ yêu cầu bạn đăng nhập vào tài khoản Microsoft
- Làm theo hướng dẫn để xác thực (thường là quét mã QR hoặc nhập mã xác thực)
- Sau khi xác thực thành công, bạn sẽ nhận được một URL tunnel và mã để kết nối

## Bước 5: Kết nối từ máy cá nhân đến server thông qua VS Code Tunnel

- Mở VS Code trên máy tính cá nhân của bạn
- Nhấn vào biểu tượng Remote Explorer trong thanh bên trái (hoặc nhấn Ctrl+Shift+P, gõ "Remote-SSH: Connect to Host")
- Trong Remote Explorer, chọn phần "Tunnels"
- Bạn sẽ thấy tunnel đã tạo từ server. Nhấp vào để kết nối
- Nếu không thấy tunnel, bạn có thể cần đăng nhập vào cùng tài khoản Microsoft đã sử dụng trên server

## Bước 6: Sử dụng VS Code Tunnel

- Sau khi kết nối thành công, bạn có thể mở thư mục từ server và làm việc như thể đang làm việc trên máy tính cá nhân
- Bạn có thể truy cập terminal, debug, sử dụng extensions,... tương tự như khi làm việc trực tiếp trên server

## Lưu ý quan trọng

1. VS Code Tunnel yêu cầu tài khoản Microsoft để xác thực
2. Máy chủ phải có kết nối internet để thiết lập tunnel
3. Nếu bạn đang sử dụng tường lửa, cần đảm bảo rằng các kết nối đi ra (outbound) được cho phép
4. Để duy trì kết nối tunnel ngay cả khi phiên SSH kết thúc, bạn có thể sử dụng `nohup` hoặc `screen`/`tmux`:
   ```bash
   nohup ./code tunnel &
   ```

## Tài liệu tham khảo

- [Tài liệu chính thức VS Code Remote Development](https://code.visualstudio.com/docs/remote/tunnels)
- [Hướng dẫn sử dụng VS Code CLI](https://code.visualstudio.com/docs/editor/command-line)
