# BÀI TẬP 1: KHỞI TẠO DROPLET TRÊN DIGITALOCEAN

> **Đường dẫn nộp bài chính thức theo yêu cầu:** [homework/session_02/ex1/README.md](file:///c:/IT_209/Session1__Bai1/homework/session_02/ex1/README.md)

Nội dung chi tiết của bài tập, log kết nối thực tế, quy trình tạo Droplet, cấu hình SSH Ed25519 và hướng dẫn kiểm tra được lưu đầy đủ tại:
👉 **[Xem chi tiết tại homework/session_02/ex1/README.md](file:///c:/IT_209/Session1__Bai1/homework/session_02/ex1/README.md)**

---

### Tóm tắt nhanh:
1. **Khóa SSH đã tạo trên máy cá nhân:**
   - Private key: `C:\Users\ADMIN\.ssh\id_ed25519`
   - Public key: `C:\Users\ADMIN\.ssh\id_ed25519.pub`
   - Thuật toán: **Ed25519**
2. **Cấu hình Droplet DigitalOcean:**
   - Region: **Singapore (SGP1)**
   - OS: **Ubuntu 22.04 LTS (x64)**
   - Plan: **Basic Plan (Shared CPU - Regular SSD)**
   - Authentication: **SSH Key**
3. **Lệnh kết nối:**
   ```bash
   ssh -i ~/.ssh/id_ed25519 root@<IP_ADDRESS_DROPLET>
   ```
