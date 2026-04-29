# Tạo thư mục

- Chạy lệnh sau để tạo thư mục odoo-erp tại thư mục gốc của dự án

  ```text
  cd ~/my-homelab-project
  mkdir odoo-erp
  cd odoo-erp
  ```
  <img width="644" height="164" alt="image" src="https://github.com/user-attachments/assets/63e66c81-63ac-4080-a958-1744106b878b" />

# Cấu hình và khởi chạy docker-compose.yml

- Tạo file .env: chạy lệnh `nano .env` để tạo file và mở ra cửa sổ nano, thêm vào đó nội dung:

  ```text
  # Database Credentials
  DB_USER=YOUR_USER
  DB_PASSWORD=YOUR_PASSWORD
  ```
- Tạo .gitignore: chạy lệnh `nano .gitignore` và thêm nội dung là `.env` để tránh push file `.env` lên mạng

- Chạy lệnh `nano docker-compose.yml` để tạo và cấu hình docker-compose.yml. Khi của sổ nano hiện lên thêm code rồi bấm Ctrl + O để lưu và Ctrl + X để thoát

  <img width="1239" height="832" alt="image" src="https://github.com/user-attachments/assets/add4cc76-3cbb-4898-b1a9-2443acb8315a" />

- `depends_on: - db` Dòng này đảm bảo rằng Docker sẽ hiểu là phải đợi máy chủ Database khởi động xong hoàn toàn rồi mới được phép bật Web Odoo lên.

- Cấp quyền đọc/ghi tối đa (777) cho thư mục data của Odoo bằng lệnh `sudo chmod -R 777 odoo_data config addons`

- Chạy lệnh `docker compose up -d` để khởi chạy

  <img width="1345" height="199" alt="image" src="https://github.com/user-attachments/assets/1332bfde-acf0-4e3f-b763-b9a979a820e5" />

# Kết quả 

- Vào trình duyệt truy cập vào địa chỉ ip 192.168.153.128:8069 để thấy giao diện của odoo

  <img width="1681" height="1026" alt="image" src="https://github.com/user-attachments/assets/07a9b84f-2c94-4dc1-a26f-4655423b902e" />

- Tại giao diện odoo cấu hình Master Password; Database Name; Email: Đây sẽ là Tên đăng nhập (Username) và Mật khẩu (Password), và cuối cùng bấm Create database để tạo cơ sở dữ liệu

  <img width="1681" height="1026" alt="Screenshot 2026-04-28 191822" src="https://github.com/user-attachments/assets/db676062-753b-4736-a31e-f5d16af2dbcd" />

- Giao diện sau khi đăng nhập thành công

  <img width="1904" height="1024" alt="image" src="https://github.com/user-attachments/assets/ad242cec-5806-4c9f-97fa-04a129e8384c" />

# Smart E-commerce

## Kích hoạt mảng Bán hàng trực tuyến (eCommerce)

- Bấm vào thanh tìm kiếm rồi tìm 2 module cốt lõi sau: eCommerce (Thương mại điện tử) và Inventory (Kho vận) và bấm nút ACTIVATE

  <img width="1918" height="972" alt="image" src="https://github.com/user-attachments/assets/4fd53ce0-b3f8-43ef-89ae-8297613338c5" />

## Thiết lập Giao diện Cửa hàng

- Bấm Let's do it để bắt đầu thiết lập 1 website

  <img width="1919" height="1006" alt="image" src="https://github.com/user-attachments/assets/1cf62fce-06b0-4c53-b5ae-2ef44c61584b" />

- Lựa chọn chủ đề website. Ở đây em muốn làm 1 website bán đồ công nghệ nên sẽ chọn **an online store (Một cửa hàng trực tuyến)** và **Electronics Store** và với mục tiêu bán được nhiều hơn nên sẽ chọn **sell more**
  
  <img width="1873" height="948" alt="image" src="https://github.com/user-attachments/assets/e88ef1d5-f29d-451b-970d-80081ae41a87" />

- Sau đó sẽ hiện ra phần chọn bảng màu (Color Palette), chọn màu yêu thích. Hệ thống sẽ hỏi muốn thêm trang phụ nào không. Em tích chọn thêm trang **About us**

  <img width="1870" height="954" alt="image" src="https://github.com/user-attachments/assets/f6499075-c57b-4c03-930c-56b819a4c82b" />

- Chọn Giao diện (Choose your Theme): Odoo sẽ render ra 3 bản giao diện khác nhau dựa trên những gì vừa chọn. Chọn cái thích nhất

  <img width="1886" height="999" alt="image" src="https://github.com/user-attachments/assets/affe618f-7dc0-45fa-ac61-78d0cbc0cf69" />

- Quá trình odoo build giao diện web

  <img width="1917" height="1004" alt="image" src="https://github.com/user-attachments/assets/f3cd53a8-6220-4b92-94a8-b843587aa76f" />

- Giao diện web khi đã build xong

  <img width="1919" height="1031" alt="Screenshot 2026-04-29 134510" src="https://github.com/user-attachments/assets/35ef2957-8d3e-40f5-be1d-06359ebe5354" />

## Kích hoạt mảng Quản trị Kho bãi (Inventory)

- Sau khi trang web đã được tạo xong tại góc trên cùng bên trái màn hình, bấm vào biểu tượng Menu (hình 9 ô vuông nhỏ) và chọn lại mục Apps (Ứng dụng) để quay về màn hình kho ứng dụng. 

  <img width="1902" height="993" alt="image" src="https://github.com/user-attachments/assets/b9906bd9-5a12-45e4-853a-21802409c795" />

- Chọn ACTIVATE ở ô Inventory.

  <img width="1912" height="934" alt="image" src="https://github.com/user-attachments/assets/c5b44be9-817e-4957-aba4-63ecf4995743" />

## Thiết lập Luồng Dữ liệu Bán hàng & Tồn kho (Sales & Inventory Data Flow)

### Đi đến Kho hàng (Inventory)

- Chọn vào biểu tượng 9 ô vuông ở góc trên cùng bên trái rồi chọn ứng dụng Inventory (Kho vận)

  <img width="1919" height="1026" alt="image" src="https://github.com/user-attachments/assets/31f29465-cf72-4fb9-b537-3d6c7951af75" />

### Tạo một Sản phẩm Công nghệ

- Trên thanh menu ngang của Inventory chọn Products (Sản phẩm) ➔ Products (Sản phẩm)

  <img width="1902" height="950" alt="image" src="https://github.com/user-attachments/assets/6319b4ef-3a68-4208-8750-fe9068c25e79" />

- Bấm nút NEW (Tạo mới) ở góc trái
 
  <img width="1862" height="964" alt="image" src="https://github.com/user-attachments/assets/76ca77e3-697f-4a19-8ddb-0be1ed3d2d54" />

- Điền thông tin:

  - Product Name (Tên sản phẩm): Thêm tên sản phẩm muốn bán
 
  - Product Type (Loại sản phẩm): phải chọn là Storable Product (Sản phẩm lưu kho)
 
  - Sales Price (Giá bán): Đặt mức giá của sản phẩm đó
 
    <img width="1895" height="1020" alt="image" src="https://github.com/user-attachments/assets/9cde66dc-1e82-475e-8d33-a3e5047ab61c" />

- Nhập kho số lượng: tại màn hình của sản phẩm vừa tạo ở lên phía trên cùng, có một hàng các nút bấm vào nút Update Quantity (Cập nhật số lượng) ➔ bấm NEW (Mới) ➔ Ở cột Counted Quantity (Số lượng thực tế) gõ vào số lượng để thể hiện số lượng đang có trong kho ➔ bấm nút Apply (Áp dụng) hoặc Save (Lưu)

  <img width="1919" height="967" alt="image" src="https://github.com/user-attachments/assets/bcff9f0a-4aa2-4965-ae77-d8815c322816" />

  <img width="1919" height="1031" alt="image" src="https://github.com/user-attachments/assets/8653b886-30e1-4467-8d56-28b64c28d00a" />

### Đăng bán lên website

- Nhấn vào tên sản phẩm (ở thanh điều hướng phía trên) để quay lại trang chi tiết sản phẩm

  <img width="1904" height="944" alt="image" src="https://github.com/user-attachments/assets/64091565-5bfa-4def-9eeb-7cdd22bdfbbc" />

- Tại góc trên cùng có một nút có biểu tượng quả cầu ghi là Go to Website (Đến trang web)

  <img width="1871" height="909" alt="image" src="https://github.com/user-attachments/assets/3bb533ec-1612-443a-9eb3-c306891389e0" />

- Hệ thống sẽ đi đến giao diện Web của sản phẩm này. Tại góc trên cùng bên phải sẽ thấy một nút gạt đang màu Đỏ ghi là Unpublished (Chưa xuất bản) ➔ gạt nó sang màu Xanh lá (Published).

  <img width="1911" height="989" alt="image" src="https://github.com/user-attachments/assets/da198237-22d6-4af2-95ed-4627cdef60bb" />

  <img width="1919" height="998" alt="image" src="https://github.com/user-attachments/assets/7f6c4e5a-9489-407d-939f-afacef1ce6ea" />

### Kích hoạt một phương thức thanh toán

- Tìm menu Thanh toán: vào biểu tượng 9 ô vuông ➔ Chọn module Website. Tại thanh menu ngang, chọn Configuration (Cấu hình) ➔ Chọn Payment Providers (Nhà cung cấp thanh toán)

  <img width="1919" height="1027" alt="image" src="https://github.com/user-attachments/assets/267c9026-ba7e-48dc-b87b-f192c55b4028" />

- Kích hoạt Chuyển khoản ngân hàng: bấm vào dòng Wire Transfer (Chuyển khoản ngân hàng). Chuyển sang tab Messages (Tin nhắn) và gõ 1 số tài khoản vào đó (Ví dụ: Vietcombank - 123456789 - Nguyễn Mạnh Hiếu)

  <img width="1919" height="997" alt="image" src="https://github.com/user-attachments/assets/5f732565-6d67-467a-ac5d-4d525787f68c" />
  
## Kiểm thử

- Đóng vai Khách hàng: Mở một cửa sổ trình duyệt Ẩn danh rồi truy cập cửa hàng bằng cách gõ vào thanh địa chỉ `http://192.168.153:8069/shop`

  <img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/abe31b1a-ad36-42b0-825f-b78169f7b7f3" />

- Đặt hàng

  <img width="1919" height="1033" alt="image" src="https://github.com/user-attachments/assets/bdae23fa-7b19-4db2-a4c4-d7a10df91647" />

  <img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/887d8f91-fe43-4b62-b95a-3e58d2a14d5f" />

- Đặt hàng thành công

  <img width="1919" height="1027" alt="image" src="https://github.com/user-attachments/assets/be7f2834-ad77-4dc0-94c0-7f211a01451a" />

- Kiểm tra đơn đặt hàng: Tại thanh menu ngang chọn eCommerce ➔ Orders (Đơn hàng)

  <img width="1918" height="997" alt="Screenshot 2026-04-29 151146" src="https://github.com/user-attachments/assets/352427e7-01c4-401b-ac4a-85fdffa0a20c" />

- Tại trang Order xóa phễu Confirmed trên thanh tìm kiếm sẽ thấy đơn S00043, bấm vào đơn hàng đó

  <img width="1914" height="996" alt="image" src="https://github.com/user-attachments/assets/db01e4fb-5c6b-4616-b570-b7f53c2f25c6" />

- Xác nhận đơn hàng: bên trong chi tiết đơn hàng S00043 sẽ có một nút màu tím tên là CONFIRM (Xác nhận), bấm CONFIRM

  <img width="1916" height="1016" alt="image" src="https://github.com/user-attachments/assets/199c6aa9-4264-4dc8-8279-aece15e640de" />

- Giao hàng cho khách (Delivery): Ngay khi bấm Confirm một nút hình chiếc xe tải nhỏ tên là Delivery (Giao hàng) sẽ xuất hiện ở góc trên bên phải. Bấm vào nút xe tải đó sẽ được đưa đến kho giao hàng. Tại đây bấm nút VALIDATE (Xác nhận giao) và hệ thống hỏi có muốn áp dụng số lượng không, chọn Apply

  <img width="1914" height="1021" alt="image" src="https://github.com/user-attachments/assets/fa99fa8e-0106-4de5-a8f4-19de01442b40" />

- Quay lại ứng dụng Inventory ➔ Products. Khi chưa xác nhận giao hàng cho khách đây là trạng thái của kho

  <img width="1918" height="959" alt="Screenshot 2026-04-29 151958" src="https://github.com/user-attachments/assets/cb3237b3-8fb2-4162-bb77-f5262f45ef2e" />

- Và đây là trạng thái của kho khi đã xác nhận giao hàng

  <img width="1912" height="935" alt="Screenshot 2026-04-29 152251" src="https://github.com/user-attachments/assets/0c7bc1a3-5e09-4910-940e-46d09d883324" />

## Logic trừ Tồn kho (On Hand vs Forecasted) của Odoo

Một trong những ưu điểm lớn nhất của hệ thống Odoo ERP là cơ chế quản lý kho thông minh thông qua hai chỉ số độc lập: On Hand và Forecasted. Dưới đây là phân tích luồng dữ liệu khi có một đơn hàng phát sinh trên Website

### Khái niệm cốt lõi

- On Hand (Tồn kho thực tế): Là số lượng vật lý của sản phẩm hiện đang nằm trên kệ kho

- Forecasted (Tồn kho khả dụng/Dự kiến): Là số lượng thực tế có thể dùng để bán cho các khách hàng tiếp theo

### Luồng thay đổi trạng thái khi khách đặt 1 sản phẩm

- Trạng thái 1: Khách vừa đặt hàng thành công (chưa xác nhận thanh toán)

  - On Hand = 20, Forecasted = 20: Đơn hàng lúc này mới ở dạng báo giá. Kế toán chưa xác nhận tiền nên thủ kho chưa làm gì cả
 
- Trạng thái 2: Kế toán bấm Xác nhận đơn (Confirm Order) - Chưa giao hàng

  - On Hand = 20, Forecasted = 19: Hàng vẫn đang nằm vật lý ở trong kho nên On Hand giữ nguyên. Tuy nhiên, hệ thống đã giữ chỗ 1 sản phẩm cho đơn hàng này. Do đó, Forecasted tụt xuống 19 để báo hiệu cho bộ phận Sale biết rằng họ chỉ còn tối đa 19 cái để bán cho khách khác, tránh tình trạng bán trùng 1 sản phẩm cho 2 người.

- Trạng thái 3: Thủ kho bấm Xác nhận giao hàng (Validate Delivery)

  - On Hand = 19, Forecasted = 19: Kiện hàng chính thức rời khỏi kho vật lý, giao cho đơn vị vận chuyển. Lúc này On Hand bị trừ đi 1, cân bằng lại với Forecasted, hoàn tất toàn bộ chu trình bán hàng - xuất kho
