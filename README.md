# Bài 2 — Phân quyền nhóm và sudoers bằng visudo

## Mục tiêu

Tạo nhóm `devops-admin` và tài khoản `deployer`, sau đó chỉ cho nhóm này quản lý trạng thái dịch vụ bằng `systemctl` mà không phải nhập mật khẩu sudo.

> Thực hiện trên Linux với một tài khoản đã có quyền `sudo`. Không sửa trực tiếp `/etc/sudoers`; luôn dùng `visudo` để kiểm tra cú pháp trước khi lưu.

## Nhật ký lệnh thực hiện

```bash
# 1. Tạo nhóm và người dùng (bỏ qua groupadd/adduser nếu đã tồn tại)
sudo groupadd devops-admin
sudo adduser deployer
sudo usermod -aG devops-admin deployer

# 2. Kiểm tra deployer đã thuộc nhóm
id deployer

# 3. Mở sudoers an toàn
sudo visudo
```

Thêm dòng sau ở cuối `/etc/sudoers` rồi lưu:

```sudoers
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Xác định chính xác vị trí `systemctl` trước khi cấu hình nếu cần:

```bash
command -v systemctl
```

## Kiểm tra bằng tài khoản deployer

```bash
su - deployer
sudo -l
sudo systemctl restart cron
```

Kết quả mẫu của `sudo -l`:

```text
User deployer may run the following commands on <hostname>:
    (ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Lệnh `sudo systemctl restart cron` hoàn tất mà không hỏi mật khẩu. Tên dịch vụ có thể là `cron` trên Ubuntu/Debian; hãy thay bằng dịch vụ đang tồn tại trên hệ thống nếu cần.

## Lưu ý an toàn

Quy tắc này chỉ cấp bốn thao tác `start`, `stop`, `restart`, và `status`; không cấp quyền chạy `systemctl` tùy ý hay quyền sudo toàn phần. Sau khi thêm user vào nhóm, hãy đăng xuất/đăng nhập lại (hoặc dùng `su - deployer`) để phiên làm việc nhận group mới.
