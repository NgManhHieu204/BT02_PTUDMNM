# Tạo thư mục

- Chạy lệnh sau để tạo thư mục odoo-erp tại thư mục gốc của dự án

  ```text
  cd ~/my-homelab-project
  mkdir odoo-erp
  cd odoo-erp
  ```
  <img width="644" height="164" alt="image" src="https://github.com/user-attachments/assets/63e66c81-63ac-4080-a958-1744106b878b" />

- Chạy lệnh `nano docker-compose.yml` để tạo và cấu hình docker-compose.yml. Khi của sổ nano hiện lên thêm code rồi bấm Ctrl + O để lưu và Ctrl + X để thoát

  <img width="1339" height="840" alt="image" src="https://github.com/user-attachments/assets/cba8f6fe-50ea-4f66-afad-bfb5bf855457" />

- `depends_on: - db` Dòng này đảm bảo rằng Docker sẽ hiểu là phải đợi máy chủ Database khởi động xong hoàn toàn rồi mới được phép bật Web Odoo lên.

- Chạy lệnh `docker compose up -d` để khởi chạy
