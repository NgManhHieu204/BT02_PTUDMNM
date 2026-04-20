# Cấu trúc thư mục triển khai (Directory Structure)

Hệ thống được tổ chức theo kiến trúc container hóa, trong đó các service được định nghĩa thông qua Docker Compose 
và dữ liệu được tách riêng bằng cơ chế volume để đảm bảo tính persistent(tính bền vững dữ liệu) và khả năng mở rộng.
```text
file-automation/
├── docker-compose.yml  # Định nghĩa các service và kiến trúc hệ thống (Docker Compose)
├── filebrowser.db      # Cơ sở dữ liệu SQLite lưu cấu hình và metadata của File Browser
├── shared_data/        # Thư mục dùng chung giữa các service (Docker Volume - Shared Data)
├── nodered_data/       # Dữ liệu persistent của Node-RED (flows, credentials, cấu hình)
└── node-red-custom/
    └── Dockerfile      # Dockerfile dùng để build custom Node-RED image
```
# Tạo cấu trúc thư mục

- Tạo và vào thư mục gốc của dự án Homelab
`mkdir my-homlab-project
cd my-homelab-project`

- Tạo và đi vào thư mục file-automation
`mkdir file-automation
cd file-automation`

- Tạo các thư mục chứa dữ liệu và thư mục build image
`mkdir shared_data` `mkdir nodered_data` `mkdir node-red-custom`

- Tạo file database rỗng cho File Browser
`touch filebrowser.db`

- Cấp quyền Đọc/Ghi để tránh lỗi Permission Denied giữa 2 container
`sudo chmod 777 shared_data nodered_data filebrowser.db`

  <img width="1311" height="301" alt="image" src="https://github.com/user-attachments/assets/02f1f9b1-9527-4aa6-914c-8931b9a082be" />

# Tạo Dockerfile

- Gói sẵn thư viện watchdirectory theo dõi file và thư viện telegrambot vào trong image

- Mở nano Dockerfile
`nano node-red-custom/Dockerfile`

- Cấu hình Dockerfile
```text
FROM nodered/node-red:latest
RUN npm install node-red-contrib-watchdirectory node-red-contrib-telegrambot
```

  <img width="978" height="374" alt="image" src="https://github.com/user-attachments/assets/2defb4f2-c971-4995-bdde-93603dab4961" />

# Khởi tạo mạng ngoại vi

- Do hệ thống được thiết kế theo kiến trúc Microservices tách rời, các dịch vụ cần một mạng dùng chung (External Network) để giao tiếp.
Nếu không có mạng này, Docker sẽ báo lỗi không tìm thấy `homelab_net`.

- Chạy lệnh: `docker network create homelab_net`

# Cấu hình và khởi chạy docker-compose.yml

- Mở docker-compose.yml: `nano docker-compose.yml`

- Cấu hình docker-compose.yml:

  <img width="1098" height="727" alt="image" src="https://github.com/user-attachments/assets/746a2970-1970-43e6-b938-681c4f78a259" />

- Khời chạy: `docker compose up -d --build`

  <img width="1473" height="584" alt="image" src="https://github.com/user-attachments/assets/91307fd0-7332-4be4-b483-beb52184ff1b" />

# Kết quả

- Truy cập địa chỉ 192.168.153.128:8080 sẽ thấy giao diện của file browser, tài khoản mặc định là admin và mật khẩu là 1 chuỗi ngẫu nhiên sinh ra,
xem bằng cách chạy lệnh `docker logs filebrowser_app` 

<img width="1877" height="1018" alt="image" src="https://github.com/user-attachments/assets/e2d36495-6799-4253-bfbb-d57214bc79fa" />

- 
