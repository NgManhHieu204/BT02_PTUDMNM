# Tạo thư mục

- Chạy lệnh sau để tạo thư mục odoo-erp tại thư mục gốc của dự án

  ```text
  cd ~/my-homelab-project
  mkdir odoo-erp
  cd odoo-erp
  ```
  <img width="644" height="164" alt="image" src="https://github.com/user-attachments/assets/63e66c81-63ac-4080-a958-1744106b878b" />

# Cấu hình và khởi chạy docker-compose.yml
  
- Chạy lệnh `nano docker-compose.yml` để tạo và cấu hình docker-compose.yml. Khi của sổ nano hiện lên thêm code rồi bấm Ctrl + O để lưu và Ctrl + X để thoát

  <img width="1239" height="832" alt="image" src="https://github.com/user-attachments/assets/add4cc76-3cbb-4898-b1a9-2443acb8315a" />

- `depends_on: - db` Dòng này đảm bảo rằng Docker sẽ hiểu là phải đợi máy chủ Database khởi động xong hoàn toàn rồi mới được phép bật Web Odoo lên.

- Cấp quyền đọc/ghi tối đa (777) cho thư mục data của Odoo bằng lệnh `sudo chmod -R 777 odoo_data config addons`

- Chạy lệnh `docker compose up -d` để khởi chạy

  <img width="1345" height="199" alt="image" src="https://github.com/user-attachments/assets/1332bfde-acf0-4e3f-b763-b9a979a820e5" />

- Kết quả: vào trình duyệt truy cập vào địa chỉ ip 192.168.153.128:8069 để thấy giao diện của odoo

  <img width="1681" height="1026" alt="image" src="https://github.com/user-attachments/assets/07a9b84f-2c94-4dc1-a26f-4655423b902e" />

- Tại giao diện odoo cấu hình Master Password; Database Name; Email: Đây sẽ là Tên đăng nhập (Username) và Mật khẩu (Password), tích chọn Demo data để Odoo sẽ tự động tạo sẵn sản phẩm, khách hàng, và đơn hàng ảo; và cuối cùng bấm Create database để tạo cơ sở dữ liệu

  <img width="1681" height="1026" alt="Screenshot 2026-04-28 191822" src="https://github.com/user-attachments/assets/db676062-753b-4736-a31e-f5d16af2dbcd" />

- Giao diện sau khi đăng nhập thành công

  <img width="1904" height="1024" alt="image" src="https://github.com/user-attachments/assets/ad242cec-5806-4c9f-97fa-04a129e8384c" />

