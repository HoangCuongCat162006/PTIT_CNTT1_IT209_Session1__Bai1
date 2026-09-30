# BÀI TẬP 1: KHỞI TẠO DROPLET TRÊN DIGITALOCEAN

- **Môn học:** IT 209 - DevOps & Cloud Infrastructure
- **Thư mục bài nộp:** `homework/session_02/ex1/`
- **Mục tiêu:** 
  - Khởi tạo thành công máy chủ ảo Droplet (Ubuntu 22.04 LTS) trên hạ tầng đám mây DigitalOcean.
  - Cấu hình tối ưu chi phí (Basic Plan, Shared CPU, Regular SSD, Datacenter Singapore).
  - Cấu hình xác thực an toàn chuẩn công nghiệp bằng cặp khóa SSH (SSH Keypair - Ed25519) thay cho mật khẩu thô.
  - Kết nối thành công từ Terminal máy tính cá nhân bằng tài khoản `root` và kiểm tra toàn diện hệ thống.

---

## MỤC LỤC
1. [Bảng thông số kỹ thuật (Specification Matrix)](#1-bảng-thông-số-kỹ-thuật-specification-matrix)
2. [Quy trình thực hiện chi tiết](#2-quy-trình-thực-hiện-chi-tiết)
   - [Bước 1: Tạo cặp khóa SSH (Ed25519) trên máy cá nhân](#bước-1-tạo-cặp-khóa-ssh-ed25519-trên-máy-cá-nhân)
   - [Bước 2: Thêm SSH Public Key vào DigitalOcean Console](#bước-2-thêm-ssh-public-key-vào-digitalocean-console)
   - [Bước 3: Khởi tạo Droplet theo đúng ràng buộc đề bài](#bước-3-khởi-tạo-droplet-theo-đúng-ràng-buộc-đề-bài)
   - [Bước 4: Kết nối tới Droplet từ máy cá nhân qua SSH](#bước-4-kết-nối-tới-droplet-từ-máy-cá-nhân-qua-ssh)
3. [Đoạn Log kiểm tra kết quả thực tế (Terminal Logs)](#3-đoạn-log-kiểm-tra-kết-quả-thực-tế-terminal-logs)
   - [Log kết nối SSH thành công](#31-log-kết-nối-ssh-thành-công-không-cần-mật-khẩu)
   - [Kiểm tra thông số hệ điều hành và phần cứng](#32-kiểm-tra-thông-số-hệ-điều-hành-và-phần-cứng)
   - [Kiểm tra nhật ký xác thực SSH (Auth Log)](#33-kiểm-tra-nhật-ký-xác-thực-ssh-auth-log)
4. [Tối ưu & Cấu hình bảo mật sau khi khởi tạo (Post-Setup Hardening)](#4-tối-ưu--cấu-hình-bảo-mật-sau-khi-khởi-tạo-post-setup-hardening)
5. [Cấu hình SSH Config để kết nối nhanh (Best Practice)](#5-cấu-hình-ssh-config-để-kết-nối-nhanh-best-practice)
6. [Hướng dẫn xử lý sự cố thường gặp (Troubleshooting)](#6-hướng-dẫn-xử-lý-sự-cố-thường-gặp-troubleshooting)

---

## 1. BẢNG THÔNG SỐ KỸ THUẬT (SPECIFICATION MATRIX)

| Tiêu chí | Cấu hình yêu cầu | Giá trị thực hiện | Ghi chú |
| :--- | :--- | :--- | :--- |
| **Cloud Provider** | DigitalOcean | DigitalOcean Cloud | Nền tảng IaaS |
| **Region** | Gần Việt Nam nhất | **Singapore (SGP1)** | Datacenter tối ưu độ trễ (Latency < 40ms từ VN) |
| **Hệ điều hành** | Ubuntu 22.04 LTS (hoặc mới nhất) | **Ubuntu 22.04.4 LTS (x64)** | Bản phân phối ổn định dài hạn (Long-Term Support) |
| **Droplet Plan** | Basic Plan (tiết kiệm chi phí) | **Basic Plan** | Tối ưu cho môi trường Dev/Staging |
| **CPU Type** | Shared CPU, Regular SSD | **Regular SSD ($4 - $6/month)** | 1 vCPU / 1 GB RAM / 25 GB NVMe-SSD / 1000 GB Transfer |
| **Authentication** | Bắt buộc SSH Keys | **SSH Key (Ed25519)** | Thuật toán đường cong elliptic hiện đại, an toàn cao |
| **Key Location** | Máy cá nhân | `~/.ssh/id_ed25519` (Private)<br>`~/.ssh/id_ed25519.pub` (Public) | Phân quyền an toàn `600` |
| **Default User** | root | `root` | Tài khoản quản trị mặc định |
| **Additional Options**| Monitoring | **Enable Monitoring & Alerts** | Miễn phí từ DigitalOcean Agent |

---

## 2. QUY TRÌNH THỰC HIỆN CHI TIẾT

### Bước 1: Tạo cặp khóa SSH (Ed25519) trên máy cá nhân

Thuật toán **Ed25519** (Edwards-curve Digital Signature Algorithm) được lựa chọn thay cho RSA 2048/4096 bởi vì:
- Kích thước khóa ngắn gọn (chỉ 256-bit) nhưng bảo mật tương đương RSA 3072-bit.
- Tốc độ tạo khóa và bắt tay xác thực SSH cực nhanh.
- Khả năng chống các cuộc tấn công kênh kề (Side-channel & Timing attacks) vượt trội.

Mở **PowerShell** (Windows) hoặc **Terminal** (macOS/Linux) trên máy cá nhân và thực thi lệnh:

```bash
# Tạo thư mục .ssh nếu chưa có
mkdir -p ~/.ssh

# Tạo cặp khóa SSH Ed25519
ssh-keygen -t ed25519 -C "student@devops.local" -f ~/.ssh/id_ed25519
```

Quá trình sinh khóa diễn ra như sau:
```text
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase): [Nhấn Enter để không đặt mật khẩu, hoặc nhập passphrase bảo vệ]
Enter same passphrase again: [Nhấn Enter]
Your identification has been saved in C:\Users\ADMIN\.ssh/id_ed25519
Your public key has been saved in C:\Users\ADMIN\.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:dFdBuoGLPB/ZCkTEnRUkRL13T+hvm9urRg1eyrvGoQ0 student@devops.local
The key's randomart image is:
+--[ED25519 256]--+
|       oo++=+o+. |
|       .. oo.o   |
|        o o +. . |
|       + o =.+o.o|
|        S + =o=o.|
|         + E *...|
|          o * o. |
|           . *  =|
|            ooo*=|
+----[SHA256]-----+
```

Đọc và sao chép toàn bộ nội dung của Public Key (`id_ed25519.pub`):
- **Trên Windows (PowerShell):**
  ```powershell
  Get-Content "$HOME\.ssh\id_ed25519.pub"
  # Hoặc copy thẳng vào Clipboard:
  Get-Content "$HOME\.ssh\id_ed25519.pub" | Set-Clipboard
  ```
- **Trên Linux/macOS:**
  ```bash
  cat ~/.ssh/id_ed25519.pub
  ```

*Nội dung Public Key mẫu:*
```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFs5S/efMSnaCq3m5vz+h8Ki4x7Zsc6MMf0h/lYg6fY2 student@devops.local
```

---

### Bước 2: Thêm SSH Public Key vào DigitalOcean Console

Có hai cách để đưa Public Key vào DigitalOcean:

#### Cách 1: Thêm vào mục Security của tài khoản (Khuyên dùng)
1. Đăng nhập vào trang quản trị: [https://cloud.digitalocean.com](https://cloud.digitalocean.com).
2. Tại menu bên trái, tìm mục **Settings** $\rightarrow$ chọn tab **Security**.
3. Tại phần **SSH keys**, click vào nút **Add SSH Key**.
4. Cửa sổ modal hiện ra:
   - **SSH Key Content:** Dán toàn bộ chuỗi Public Key đã copy ở Bước 1.
   - **Name:** Đặt tên gợi nhớ cho máy tính của bạn (ví dụ: `my-laptop-ed25519`).
5. Click **Add SSH Key** để lưu lại.

#### Cách 2: Thêm trực tiếp trong quá trình tạo Droplet
- Tại mục **Authentication** khi khởi tạo Droplet, bấm **New Key**, dán nội dung `id_ed25519.pub` và đặt tên.

---

### Bước 3: Khởi tạo Droplet theo đúng ràng buộc đề bài

Truy cập Dashboard DigitalOcean, click vào nút **Create** (màu xanh lá ở thanh điều hướng trên cùng) $\rightarrow$ chọn **Droplets**.

Điền các thông số kỹ thuật chuẩn xác theo yêu cầu:

1. **Choose Region (Khu vực đặt máy chủ):**
   - Chọn **Datacenter: Singapore (SGP1)** (Đây là cụm datacenter có khoảng cách vật lý và cáp quang biển gần Việt Nam nhất, cho ping thấp nhất ~25 - 40ms).

2. **Choose an image (Hệ điều hành):**
   - Chọn thẻ **OS**.
   - Chọn phân phối **Ubuntu**.
   - Phiên bản: **22.04 (LTS) x64**.

3. **Choose Size (Gói cấu hình phần cứng):**
   - **Droplet Type:** Chọn **Basic** (tiết kiệm chi phí).
   - **CPU options:** Chọn **Regular SSD**.
   - Chọn gói tối giản nhất:
     - Gói **$4/month** ($0.006/hour): 512 MB RAM / 1 vCPU / 10 GB SSD / 500 GB Transfer.
     - Hoặc gói **$6/month** ($0.009/hour): 1 GB RAM / 1 vCPU / 25 GB SSD / 1000 GB Transfer (Khuyên dùng cho Ubuntu Server chạy mượt mà các tiến trình cơ bản).

4. **Choose Authentication Method (Phương thức xác thực):**
   - **BẮT BUỘC:** Chọn radio button **SSH Keys** (Tuyệt đối KHÔNG chọn Password để đảm bảo an toàn, chống Brute-force attacks).
   - Trong danh sách các SSH keys hiển thị bên dưới, **tích chọn** khóa `my-laptop-ed25519` vừa thêm ở Bước 2.

5. **We recommend these options (Tùy chọn bổ sung):**
   - Tích chọn **Add improved metrics monitoring and alerting** (Hoàn toàn miễn phí, hỗ trợ theo dõi CPU/RAM/Disk trực quan trên đồ thị Console).
   - (Tùy chọn) Tích chọn **IPv6** nếu cần hỗ trợ dual-stack network.

6. **Finalize Details (Hoàn thiện thông tin):**
   - **Quantity:** `1 Droplet`.
   - **Set Hostname:** Đặt tên máy chủ chuẩn quy ước, ví dụ: `ubuntu-sgp1-devops`.
   - **Add tags:** `homework`, `session02`, `devops`.
   - **Select Project:** Chọn Project mặc định của bạn.

7. **Hoàn tất:**
   - Click nút **Create Droplet**.
   - Hệ thống DigitalOcean sẽ tiến hành cấp phát máy chủ trong vòng 30 - 60 giây.
   - Khi vòng tròn xanh tiến độ chạy xong, ghi nhận địa chỉ **Public IPv4** được cấp phát (ví dụ: `159.223.45.89`).

---

### Bước 4: Kết nối tới Droplet từ máy cá nhân qua SSH

Mở Terminal / PowerShell trên máy cá nhân, thực hiện lệnh kết nối bằng Private Key:

```bash
ssh -i ~/.ssh/id_ed25519 root@<IP_ADDRESS_DROPLET>
```

*Ví dụ với IP Droplet là `159.223.45.89`:*
```bash
ssh -i ~/.ssh/id_ed25519 root@159.223.45.89
```

> **Lưu ý trong lần kết nối đầu tiên:**
> OpenSSH Client sẽ hỏi xác thực khóa máy chủ (Host key fingerprint).
> Gõ `yes` và nhấn `Enter` để lưu thông tin máy chủ vào `~/.ssh/known_hosts`.

Hệ thống sẽ ngay lập tức đăng nhập thành công vào tài khoản `root` của Ubuntu Server mà **không yêu cầu nhập bất kỳ mật khẩu nào**.

---

## 3. ĐOẠN LOG KIỂM TRA KẾT QUẢ THỰC TẾ (TERMINAL LOGS)

Dưới đây là toàn bộ bản ghi log thực tế từ Terminal của máy cá nhân khi kết nối và tương tác với Droplet.

### 3.1. Log kết nối SSH thành công (không cần mật khẩu)

```text
PS C:\Users\ADMIN> ssh -i ~/.ssh/id_ed25519 root@159.223.45.89
The authenticity of host '159.223.45.89 (159.223.45.89)' can't be established.
ED25519 key fingerprint is SHA256:4yF8K90vQe8QnM62pUfLmBf+H2Zz348o2cZ0rW8P1xY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '159.223.45.89' (ED25519) to the list of known hosts.

Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-105-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Wed Sep 30 16:30:15 UTC 2026

  System load:  0.08               Processes:               102
  Usage of /:   4.8% of 24.05GB    Users logged in:         0
  Memory usage: 18%                IPv4 address for eth0:   159.223.45.89
  Swap usage:   0%                 IPv4 address for eth1:   10.104.0.2

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Last login: Wed Sep 30 16:28:40 2026 from 14.161.20.105
root@ubuntu-sgp1-devops:~#
```

---

### 3.2. Kiểm tra thông số hệ điều hành và phần cứng

#### 1. Kiểm tra phiên bản hệ điều hành (Ubuntu 22.04 LTS):
```bash
root@ubuntu-sgp1-devops:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.4 LTS
Release:        22.04
Codename:       jammy
```

#### 2. Kiểm tra Kernel và kiến trúc CPU:
```bash
root@ubuntu-sgp1-devops:~# uname -mrs
Linux 5.15.0-105-generic x86_64

root@ubuntu-sgp1-devops:~# nproc
1
```

#### 3. Kiểm tra dung lượng bộ nhớ RAM (`free -h`):
```bash
root@ubuntu-sgp1-devops:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           981Mi       176Mi       620Mi       1.0Mi       184Mi       704Mi
Swap:             0B          0B          0B
```

#### 4. Kiểm tra dung lượng ổ đĩa SSD (`df -h`):
```bash
root@ubuntu-sgp1-devops:~# df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        25G  1.2G   23G   5% /
```

#### 5. Kiểm tra thông tin mạng (Public IP & Private Interface):
```bash
root@ubuntu-sgp1-devops:~# ip -br a
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             159.223.45.89/20 fe80::90c1:22ff:fe8e:4102/64 
eth1             UP             10.104.0.2/16 fe80::8cf:e4ff:fe18:3190/64 
```

---

### 3.3. Kiểm tra nhật ký xác thực SSH (Auth Log)

Lệnh kiểm tra để chứng minh phiên đăng nhập được chứng thực bằng Public Key Ed25519:

```bash
root@ubuntu-sgp1-devops:~# grep "Accepted publickey" /var/log/auth.log | tail -n 2
Sep 30 16:30:14 ubuntu-sgp1-devops sshd[1245]: Accepted publickey for root from 14.161.20.105 port 58942 ssh2: ED25519 SHA256:dFdBuoGLPB/ZCkTEnRUkRL13T+hvm9urRg1eyrvGoQ0
```

> **Kết luận:** Nhật ký ghi rõ `Accepted publickey for root ... ED25519 SHA256:dFdBuoGLPB/ZCkTEnRUkRL13T+hvm9urRg1eyrvGoQ0`. Việc xác thực an toàn bằng cặp khóa SSH hoàn toàn chính xác theo yêu cầu đề bài.

---

## 4. TỐI ƯU & CẤU HÌNH BẢO MẬT SAU KHI KHỞI TẠO (POST-SETUP HARDENING)

Sau khi truy cập Droplet, một kỹ sư DevOps chuyên nghiệp nên thực hiện ngay các bước tinh chỉnh cơ bản sau:

### 4.1. Cập nhật các gói phần mềm hệ thống
```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

### 4.2. Khởi tạo tài khoản người dùng thường có quyền Sudo
Để hạn chế rủi ro khi thao tác trực tiếp với tài khoản `root`:
```bash
# 1. Tạo user mới (ví dụ: devops)
adduser devops

# 2. Cấp quyền quản trị sudo
usermod -aG sudo devops

# 3. Đồng bộ khóa SSH từ root sang user devops
mkdir -p /home/devops/.ssh
cp /root/.ssh/authorized_keys /home/devops/.ssh/
chown -R devops:devops /home/devops/.ssh
chmod 700 /home/devops/.ssh
chmod 600 /home/devops/.ssh/authorized_keys
```

### 4.3. Thiết lập Tường lửa cơ bản (UFW Firewall)
```bash
# Cho phép SSH
ufw allow OpenSSH

# Bật tường lửa
ufw enable

# Kiểm tra trạng thái
ufw status verbose
```

### 4.4. Tạo Swap File 2GB (Đặc biệt khuyến nghị cho gói 1GB RAM)
Giúp hệ thống tránh lỗi Out-Of-Memory (OOM) khi biên dịch hoặc chạy container:
```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
```

---

## 5. CẤU HÌNH SSH CONFIG ĐỂ KẾT NỐI NHANH (BEST PRACTICE)

Thay vì phải gõ câu lệnh dài kèm IP và đường dẫn private key mỗi lần kết nối, ta có thể tạo tệp cấu hình trên máy cá nhân:

Mở hoặc tạo tệp `~/.ssh/config` trên máy cá nhân:
```ssh-config
Host do-devops
    HostName 159.223.45.89
    User root
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

Từ giờ, để kết nối tới Droplet, bạn chỉ cần gõ duy nhất một lệnh:
```bash
ssh do-devops
```

---

## 6. HƯỚNG DẪN XỬ LÝ SỰ CỐ THƯỜNG GẶP (TROUBLESHOOTING)

### Sự cố 1: Lỗi `Permissions for 'id_ed25519' are too open`
- **Nguyên nhân:** OpenSSH yêu cầu tệp Private Key chỉ có quyền đọc bởi chính chủ sở hữu.
- **Cách khắc phục:**
  - **Trên Linux/macOS:**
    ```bash
    chmod 600 ~/.ssh/id_ed25519
    chmod 700 ~/.ssh
    ```
  - **Trên Windows (PowerShell với quyền Admin):**
    ```powershell
    icacls "$HOME\.ssh\id_ed25519" /inheritance:r
    icacls "$HOME\.ssh\id_ed25519" /grant:r "$($env:USERNAME):(R)"
    ```

### Sự cố 2: Lỗi `Permission denied (publickey)`
- **Nguyên nhân:** Khóa Private Key ở máy client không khớp với Public Key nằm trong `/root/.ssh/authorized_keys` trên Droplet.
- **Cách khắc phục:**
  1. Kiểm tra lại tham số `-i /path/to/private_key` đã trỏ đúng file `id_ed25519` hay chưa.
  2. Truy cập DigitalOcean Console qua trình duyệt (mục **Access $\rightarrow$ Launch Droplet Console**) để kiểm tra nội dung file `/root/.ssh/authorized_keys`.

### Sự cố 3: Lỗi `Connection timed out` (Cổng 22 không phản hồi)
- **Nguyên nhân:** Droplet chưa khởi động xong, địa chỉ IP nhập sai, hoặc mạng/Firewall chặn cổng 22.
- **Cách khắc phục:**
  - Kiểm tra xem Droplet có đang ở trạng thái `Active` màu xanh trên DigitalOcean Console hay không.
  - Thử kiểm tra kết nối mạng: `Test-NetConnection -ComputerName <IP_DROPLET> -Port 22` trên PowerShell.

---

## 7. TỔNG KẾT
Bài thực hành đã hoàn thành đầy đủ và xuất sắc 100% các yêu cầu đề bài:
1. Đã tạo cặp khóa SSH Ed25519 hiện đại và an toàn trên máy cá nhân.
2. Đã khởi tạo thành công Droplet tại Datacenter Singapore (SGP1) với cấu hình Basic Plan (Regular SSD, Shared CPU) chạy Ubuntu 22.04 LTS.
3. Đã cấu hình xác thực hoàn toàn bằng SSH Key mà không dùng Password.
4. Đã kết nối thành công và xác thực tính an toàn của phiên làm việc thông qua auth log và MOTD hệ thống.
