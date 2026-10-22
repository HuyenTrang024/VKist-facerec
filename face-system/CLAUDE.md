# CLAUDE.md – face-system (VKIST Face Recognition, bản Docker)

## Tổng quan

Hệ thống điểm danh bằng khuôn mặt, chạy bằng Docker Compose với 2 service:

| Service | Container | Cổng | Công nghệ |
|---|---|---|---|
| `backend` | `attendance-backend` | 5002 | aiohttp + PyTorch (ArcFace, RFB detector, Deep3D bỏ khẩu trang, DREAM), SQLite, hnswlib |
| `frontend` | `attendance-frontend` | 5000 | Flask + TensorFlow Lite, đọc webcam/RTSP, gửi crop sang backend `/facerec` |

- Máy chủ hiện tại: `itvkist-pilot-tuf`, IP LAN `10.1.8.125` (Wi-Fi, DHCP, có thể đổi).
- Người dùng máy khác chỉ cần trình duyệt: `http://10.1.8.125:5000` (camera), `:5002` (quản trị). Khác mạng: dùng Tailscale.
- Theo dõi container: Portainer (`https://localhost:9443`).

## Lệnh

```bash
sudo docker compose up -d            # chạy cả backend + frontend (image đã build)
sudo docker compose up -d --build    # build lại khi sửa code
sudo docker compose ps
sudo docker compose logs -f backend
```

User `itvkist` chưa thuộc nhóm `docker` nên phải dùng `sudo` (hoặc `sudo usermod -aG docker $USER` rồi đăng nhập lại).
Claude không có quyền gọi Docker trong phiên làm việc.

## Cấu hình & dữ liệu

- `.env`: `SECRET_KEY` (chỉ frontend dùng, lấy từ trang `:5002` sau khi đăng ký), `RTSP_URLS` (phân tách bằng dấu phẩy), `REQUEST_INTERVAL`.
- Backend **không** đọc `SECRET_KEY` từ env; key là UUID sinh khi đăng ký tài khoản.
- Dữ liệu bind mount: `data/instance` (SQLite `face_reg.db`), `data/indexes`, `data/images`. Volume `model_cache` → `/root/.cache`.
- Weights không có trong Git: `weights-backend.zip` → `backend/`, `weights-frontend.zip` → `frontend/` (bản zip nằm ở `../`).

## Lưu ý kỹ thuật

- Deep3D import `nvdiffrast.torch` và dùng `RasterizeCudaContext` → backend **bắt buộc GPU NVIDIA**; nvdiffrast JIT-compile lúc chạy lần đầu nên cần `nvcc` (image CUDA devel).
- `backend/.dockerignore` loại `env/`, `images/`, `indexes/`, `instance/`, `*.db`, `miniconda.sh`.
- Không đặt thư mục lưu Docker bên trong `backend/` hoặc `frontend/` (là build context).
- `/media/itvkist/Projects` là ổ **NTFS (fuseblk)** → Docker không lưu image trực tiếp lên đó được.

## Nhật ký 2026-10-16

### Đã thay đổi
- `backend/Dockerfile`: base `python:3.10-slim` → `nvidia/cuda:12.1.1-devel-ubuntu22.04`, cài Python 3.10 hệ thống, `git`, `ninja-build`, và `nvdiffrast@v0.3.3` (`--no-build-isolation`). Trước đó image thiếu nvdiffrast nên backend không khởi động được (cổng 5002 không listen).
- `backend/create_app.py`: DB chuyển từ `./face_reg.db` → `./instance/face_reg.db` (tạo `instance/` trước khi tạo engine) để nằm trong volume `data/instance`. `export_data.py` vốn đã đọc `instance/face_reg.db`.
- `docker-compose.yml`: bỏ `SECRET_KEY` thừa ở backend; thêm `deploy.resources.reservations.devices` (GPU nvidia); volume `torch_cache` → `model_cache` mount `/root/.cache`.
- `frontend/app.py`: thêm route `POST /facereg` proxy sang `BACKEND_URL/facereg`.
- `frontend/templates/index.html`: `BACKEND_URL` từ `"http://localhost:5002"` → `""` (gọi `/facereg` cùng origin). Sửa lỗi đăng ký khuôn mặt từ máy khác (localhost) và CORS.
- `README.md`: hiện là **hướng dẫn ngắn cho người dùng** (truy cập LAN / Tailscale). Bản README dài cho quản trị (vận hành máy chủ, Portainer, cài mới từ đầu) đã bị xóa khỏi đĩa, có thể viết lại thành `HUONG_DAN_QUAN_TRI.md` nếu cần.
- `../.gitignore`: thêm `docker-data/`, `face-system/data/`.

### Trạng thái cuối ngày (chưa xong)
- Build backend mới **thất bại**: `no space left on device` khi giải nén image CUDA devel vào `/var/lib/containerd`.
- Ổ hệ thống `/` (nvme0n1p5, ext4): 98G, đã dùng 92G, còn ~1,8G. Docker/containerd chỉ chiếm ~9G; phần lớn còn lại chưa xác định.
- Ổ Projects (nvme0n1p4, NTFS): 156G, còn 22G, không đủ cho image backend (~15G) + frontend (~3–4G) + build cache.
- Lần thử chuyển data-root bị lỗi do biến `$D` rỗng → đã hướng dẫn hoàn tác: xóa `/docker`, `/containerd`, gỡ dòng `root = "/containerd"` ở `/etc/containerd/config.toml`, xóa `/etc/docker/daemon.json`. **Cần xác nhận user đã hoàn tác** và Docker đã bật lại (`Docker Root Dir` = `/var/lib/docker`).

### Việc tiếp theo
1. Xác định thứ chiếm chỗ trên `/` (`sudo du -xh --max-depth=1 / | sort -h`), dọn để có ≥ 30G trống.
2. Nếu vẫn thiếu: chuyển `backend/Dockerfile` sang multi-stage (devel để build nvdiffrast, runtime nhẹ hơn), hoặc tắt containerd snapshotter để không lưu trùng blob nén.
3. Build lại `sudo docker compose up -d --build`, kiểm tra backend `Up`, `torch.cuda.is_available()` = True, cổng 5002 listen.
4. Mở ufw 5000/5002 nếu đang active; cài Tailscale cho truy cập ngoài LAN; điền địa chỉ `100.x.x.x` và liên hệ quản trị vào `README.md`.
5. Database cũ ở `backend/face_reg.db` (~9 MB, có tài khoản thật) cần copy vào `data/instance/face_reg.db` nếu muốn giữ.
