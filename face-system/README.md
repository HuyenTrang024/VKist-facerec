# Hệ thống điểm danh khuôn mặt VKIST – Hướng dẫn truy cập

## Giới thiệu

Hệ thống được vận hành trên máy chủ của VKIST và truy cập qua trình duyệt web. Máy người dùng không cần cài đặt phần mềm hay thư viện nào khác.

| Thành phần | Cổng | Chức năng |
|---|---|---|
| Màn hình camera | `5000` | Theo dõi camera trực tiếp, xem kết quả nhận diện, đăng ký khuôn mặt |
| Trang quản trị | `5002` | Quản lý tài khoản, danh sách nhân sự, lớp và lịch sử điểm danh |

## Yêu cầu

- Trình duyệt Google Chrome, Microsoft Edge hoặc Mozilla Firefox (phiên bản mới).
- Kết nối vào mạng nội bộ của cơ quan, hoặc cài đặt Tailscale khi truy cập từ bên ngoài.

## 1. Truy cập trong mạng nội bộ (LAN)

1. Kết nối máy tính vào Wi-Fi hoặc mạng dây của cơ quan.
2. Mở trình duyệt và truy cập:

   | Thành phần | Địa chỉ |
   |---|---|
   | Màn hình camera | http://10.1.8.125:5000 |
   | Trang quản trị | http://10.1.8.125:5002 |

Địa chỉ phải bắt đầu bằng `http://` và kèm số cổng.

## 2. Truy cập từ bên ngoài (qua Tailscale)

1. Gửi địa chỉ email cho quản trị viên để nhận lời mời, sau đó chọn **Accept invite** trong email.
2. Tải và cài đặt Tailscale tại https://tailscale.com/download (Windows, macOS, Android, iOS).
3. Mở Tailscale, chọn **Log in** và đăng nhập bằng email đã được mời.
4. Khi trạng thái hiển thị **Connected**, truy cập:

   | Thành phần | Địa chỉ |
   |---|---|
   | Màn hình camera | `http://<địa-chỉ-tailscale>:5000` |
   | Trang quản trị | `http://<địa-chỉ-tailscale>:5002` |

   `<địa-chỉ-tailscale>` có dạng `100.x.x.x` và do quản trị viên cung cấp.

## 3. Xử lý sự cố

| Sự cố | Cách khắc phục |
|---|---|
| Không thể truy cập trang | Kiểm tra địa chỉ và số cổng; kiểm tra kết nối mạng nội bộ hoặc trạng thái Tailscale. |
| Trang tải chậm hoặc không phản hồi | Chờ vài phút rồi tải lại trang. |
| Không hiển thị camera hoặc không nhận diện được | Liên hệ quản trị viên. |

## Liên hệ

Quản trị viên hệ thống: _[Họ tên] – [Số điện thoại] – [Email]_
