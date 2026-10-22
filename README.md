# VKist-facerec

Hệ thống điểm danh bằng khuôn mặt của VKIST. Hệ thống nhận diện được cả khi người dùng đeo khẩu trang, đeo kính hoặc quay mặt nghiêng.

| Thành phần | Cổng | Công nghệ | Chức năng |
|---|---|---|---|
| `backend` | 5002 | aiohttp, PyTorch, SQLite, hnswlib | Nhận diện (RFB detector, ArcFace, Deep3D bỏ khẩu trang, DREAM), trang quản trị |
| `frontend` | 5000 | Flask, TensorFlow Lite | Đọc webcam/camera RTSP, phát hiện khuôn mặt, gửi sang backend |

## Cấu trúc thư mục

```
VKist-facerec/
├── backend/
│   ├── app/                # ArcFace, RFB face detector, phân loại phụ kiện
│   ├── deep3d/             # Tái tạo 3D, bỏ khẩu trang
│   ├── face_dream/         # DREAM – hiệu chỉnh embedding theo góc mặt
│   ├── static/, templates/ # Giao diện trang quản trị
│   ├── app.py              # Server backend
│   ├── create_app.py       # Model CSDL
│   ├── export_data.py      # Xuất điểm danh ra Excel theo tháng
│   └── Dockerfile
├── frontend/
│   ├── utils/              # Detector, alignment TFLite và công cụ chuyển đổi model
│   ├── templates/          # Màn hình camera
│   ├── app.py              # Server frontend (1 camera)
│   ├── app_multi.py        # Bản nhiều camera
│   └── Dockerfile
├── tests/                  # Script thử nghiệm (Polyface, khẩu trang, kính, ánh sáng yếu...)
├── data/                   # Dữ liệu chạy: DB, index, ảnh (không commit)
└── docker-compose.yml
```

## Cài đặt

**Yêu cầu:** Linux, GPU NVIDIA, Docker và [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).

**1. Tải code và weights**

```bash
git clone https://github.com/HuyenTrang024/VKist-facerec.git
cd VKist-facerec
```

Tải `weights-backend.zip` và `weights-frontend.zip` từ [Google Drive](https://drive.google.com/drive/folders/1Nb4V75i_BX00RT0HQNAIcgsKmYLQpn9p?usp=sharing), rồi giải nén:

```bash
unzip weights-backend.zip -d backend/
unzip weights-frontend.zip -d frontend/
```

**2. Tạo file `.env`**

```env
SECRET_KEY=           # lấy ở trang quản trị sau khi đăng ký tài khoản
RTSP_URLS=            # camera IP, cách nhau bằng dấu phẩy; để trống nếu dùng webcam
REQUEST_INTERVAL=15
```

**3. Chạy**

```bash
docker compose up -d --build
docker compose logs -f backend
```

Lần chạy đầu sẽ chậm, vì nvdiffrast phải biên dịch CUDA.

**4. Lần đầu sử dụng**

1. Mở `http://localhost:5002`, đăng ký tài khoản và lấy secret key.
2. Điền secret key vào `SECRET_KEY` trong `.env`.
3. Chạy `docker compose up -d` để khởi động lại.

## Sử dụng

| Trang | Địa chỉ |
|---|---|
| Màn hình camera, đăng ký khuôn mặt | `http://<ip-máy-chủ>:5000` |
| Trang quản trị: tài khoản, nhân sự, lịch sử điểm danh | `http://<ip-máy-chủ>:5002` |

Máy khác trong cùng mạng LAN chỉ cần mở trình duyệt. Muốn truy cập từ ngoài mạng thì dùng [Tailscale](https://tailscale.com/download).

Xuất dữ liệu điểm danh ra Excel:

```bash
docker compose exec backend sh -c "pip install pandas openpyxl && python export_data.py"
docker compose cp backend:/app/exported_timeline_monthly.xlsx .
```
