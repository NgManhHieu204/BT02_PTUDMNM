# MÔN HỌC: PHÁT TRIỂN ỨNG DỤNG MÃ NGUỒN MỞ

# HỌ VÀ TÊN: NGUYỄN MẠNH HIẾU - MSSV: K225480106020 - LỚP: K58KTP

# MỤC LỤC

**[Chi tiết toàn bộ Quá trình Triển khai và Kiểm thử của File Automation Engine.](docs/file_automation.md)**

# Tổng quan

Dự án này là một hệ thống máy chủ cá nhân (Homelab) được xây dựng theo kiến trúc **Microservices phân tách độc lập**. Thay vì gom tất cả vào một cụm duy nhất, hệ thống được chia nhỏ thành các `services` riêng biệt.

## File Browser + Node-RED

- **File Browser** là một phần mềm mã nguồn mở cho phép quản lý tệp tin thông qua giao diện web. Công cụ này giúp người dùng truy cập, tải lên, tải xuống, chỉnh sửa, di chuyển và phân quyền file/thư mục ngay trên trình duyệt mà không cần thao tác trực tiếp bằng dòng lệnh trên máy chủ. Trong môi trường truyền thống, việc quản lý file trên máy chủ thường phải thực hiện qua: "SSH / Terminal", "FTP / SFTP Client", truy cập vật lý vào máy tính hoặc server. File Browser ra đời nhằm đơn giản hóa quá trình quản lý dữ liệu bằng cách cung cấp giao diện trực quan, dễ sử dụng và có thể truy cập ở bất kỳ đâu qua mạng nội bộ hoặc Internet.

- **Node-RED** là nền tảng lập trình trực quan theo mô hình luồng xử lý (Flow-based Programming), được phát triển trên nền tảng Node.js. Thay vì viết mã nguồn bằng tay thì Node-RED cho phép người dùng chỉ cần kéo thả các khối chức năng (node) và kết nối chúng để tạo thành quy trình tự động hóa.

### 1. File Automation Engine (Quản lý File & Tự động hóa)

* **Công nghệ cốt lõi:** File Browser + Node-RED + Docker.
* **Giới thiệu bài toán:** Trong môi trường doanh nghiệp hoặc hệ thống cá nhân (Homelab), việc quản lý file truyền thống thường mang tính "tĩnh" (chỉ lưu trữ và tải về). Dự án này giải quyết bài toán biến một kho lưu trữ tĩnh thành một Hệ thống Xử lý Tự động bằng cách kết hợp File Browser và Node-RED.
* **Phân tích thành phần:** Hệ thống được chia làm hai lớp rõ rệt, hoạt động độc lập nhưng giao tiếp chặt chẽ với nhau:

  * **File Browser:** Đóng vai trò là hệ thống quản lý file dựa trên nền Web, cung cấp giao diện trực quan để người dùng (End-user) có thể tải lên (Upload), tải xuống (Download), xóa, và phân quyền tài liệu.
 
  * **Node-RED:** * Đóng vai trò là hệ thống điều phối và tự động hóa, chạy ngầm phía sau để liên tục giám sát trạng thái của hệ thống file để kích hoạt các kịch bản như gửi cảnh báo, bóc tách dữ liệu mà không cần sự can thiệp thủ công của con người.
 
* **Nguyên lý hoạt động cốt lõi "Shared Volume":** Điểm cốt lõi cho phép hai hệ thống này tích hợp với nhau trong môi trường Docker container là cơ chế Shared Bind Mounts.
  * **Bản chất:** Cả File Browser và Node-RED đều được cấu hình truy cập tới cùng một thư mục vật lý trên máy chủ (ví dụ: ./shared_data). Thư mục này đóng vai trò là vùng lưu trữ chung (shared storage), cho phép các container truy cập và thao tác trên cùng một tập dữ liệu.
  * **Cách hoạt động:** Khi người dùng upload file thông qua File Browser, dữ liệu sẽ được ghi trực tiếp vào thư mục vật lý đã mount. Node-RED có quyền truy cập vào cùng thư mục này, do đó có thể phát hiện các thay đổi dữ liệu và xử lý tương ứng.
 
* **Luồng dữ liệu tự động hóa (Data Flow):** Trong kịch bản cảnh báo (Smart Alerting), luồng xử lý dữ liệu diễn ra như sau:

  * **Trigger (Kích hoạt):** Người dùng upload một tệp tin thông qua giao diện File Browser (ví dụ: bao_cao.pdf).
  * **Write (Ghi dữ liệu):** File được lưu vào thư mục vật lý trên hệ thống thông qua cơ chế mount.
  * **Listen (Giám sát):** Node-RED sử dụng node `watch` để phát hiện các sự kiện thay đổi trong thư mục (file created hoặc modified).
  * **Execute (Thực thi):** Node-RED thu thập metadata của file (tên file, thời gian tạo,...) và kích hoạt API của dịch vụ bên thứ ba (ví dụ: Telegram) để gửi thông báo đến quản trị viên.

### 2. Tích hợp Shadow Backup & Audit Trail

Trong quá trình làm bài em thấy việc chỉ cảnh báo là chưa đủ an toàn nếu người dùng vô tình hoặc cố ý xóa/sửa file quan trọng. Vì vậy, hệ thống được nâng cấp thêm tính năng **Shadow Backup (Sao lưu ngầm)** với luồng xử lý như sau:

* **Bổ sung Storage Layer:** Tạo thêm một vùng nhớ độc lập `/backup`. Vùng nhớ này chỉ Node-RED và Admin có quyền truy cập, người dùng File Browser không thể nhìn thấy để can thiệp.
* **Thêm mới:** Node-RED phát hiện file mới -> Copy file gốc sang `/backup` -> Gửi Telegram báo cáo.
* **Chỉnh sửa:** Node-RED phát hiện file bị sửa -> Copy bản sửa đó sang `/backup`, tự động gắn thêm Timestamp (Versioning) để không đè lên bản gốc -> Gửi Telegram cảnh báo.
* **Xóa:** Node-RED phát hiện file bị xóa trên File Browser -> Gửi Telegram báo động đỏ (Dữ liệu vẫn nằm an toàn trong `/backup` chờ Admin xử lý).

## Odoo ERP + PostgreSQL

- **ERP (Enterprise Resource Planning)** dịch ra nghĩa là: Hoạch định nguồn lực doanh nghiệp. Đây là hệ thống phần mềm được xây dựng nhằm tích hợp và quản lý toàn bộ hoạt động của doanh nghiệp trên một nền tảng thống nhất. Trong mô hình vận hành truyền thống, doanh nghiệp thường sử dụng nhiều công cụ riêng lẻ cho từng nghiệp vụ khác nhau như: Quản lý kho bằng Excel, Chăm sóc khách hàng qua Zalo hoặc Email,... . Việc sử dụng nhiều hệ thống tách rời dễ dẫn đến dữ liệu phân mảnh, khó đồng bộ, mất thời gian xử lý và khó kiểm soát tổng thể. ERP ra đời để giải quyết vấn đề này bằng cách tập trung toàn bộ dữ liệu và quy trình vận hành vào một hệ thống duy nhất, giúp các phòng ban phối hợp hiệu quả hơn và hỗ trợ nhà quản lý đưa ra quyết định chính xác.

- **Odoo** là một nền tảng ERP mã nguồn mở (Open Source ERP) nổi tiếng và được sử dụng rộng rãi trên thế giới. Hệ thống được phát triển bằng ngôn ngữ Python và sử dụng PostgreSQL làm cơ sở dữ liệu chính. Điểm mạnh của Odoo là kiến trúc dạng module, cho phép doanh nghiệp triển khai linh hoạt theo nhu cầu thực tế. Một số module phổ biến gồm: Bán hàng (Sales), Quản lý khách hàng (CRM), Kho vận (Inventory), Kế toán (Accounting), Nhân sự (HR), Sản xuất (Manufacturing), Website / Thương mại điện tử (E-commerce),...

- **PostgreSQL** là hệ quản trị cơ sở dữ liệu quan hệ mạnh mẽ, ổn định và bảo mật cao. Trong Odoo, PostgreSQL đóng vai trò lưu trữ toàn bộ dữ liệu nghiệp vụ như: Thông tin khách hàng, Sản phẩm, Đơn hàng, Tồn kho, Hóa đơn ,Nhân sự, Báo cáo quản trị. Sự kết hợp giữa Odoo và PostgreSQL tạo nên một giải pháp ERP mạnh, ổn định và phù hợp cho nhiều mô hình doanh nghiệp khác nhau.

### 1. Smart E-commerce ERP (Thương mại Điện tử & Quản trị Doanh nghiệp)

* **Công nghệ cốt lõi:** Odoo (Web Application), PostgreSQL (Relational Database), Docker.
* **Giới thiệu bài toán:** Đối với các doanh nghiệp vừa và nhỏ (SME) chuyên bán lẻ trực tuyến, việc sử dụng các nền tảng phân mảnh (web một nơi, quản lý kho một nơi) gây sai lệch dữ liệu và tốn kém chi phí thuê bao hàng tháng. Dự án này triển khai một hệ thống ERP mã nguồn mở tự lưu trữ (Self-hosted), gom toàn bộ luồng vận hành từ lúc khách xem hàng đến lúc xuất kho vào một cơ sở dữ liệu duy nhất.
* **Phân tích thành phần:** Hệ thống được thiết kế theo kiến trúc 2 tầng tách biệt:

  * **Odoo:** Đóng vai trò là máy chủ ứng dụng web, cung cấp giao diện Frontend (Website bán hàng cho khách) và Backend (Dashboard quản trị kho, doanh thu cho chủ doanh nghiệp).
  
  * **PostgreSQL:** Đóng vai trò là hệ quản trị cơ sở dữ liệu quan hệ, lưu trữ toàn bộ dữ liệu lõi của doanh nghiệp một cách độc lập, đảm bảo an toàn và dễ dàng sao lưu định kỳ.

* **Nguyên lý hoạt động cốt lõi "Kiến trúc Module (Modular Architecture)":**

  * **Bản chất:** Odoo khởi chạy như một bộ khung lõi (Core Framework). Các tính năng nghiệp vụ được cài đặt dưới dạng các Module độc lập (Website, eCommerce, Inventory, Invoicing).
  * **Cách hoạt động:** Các module sử dụng chung một cấu trúc dữ liệu quan hệ (Relational Data). Sự kiện phát sinh ở module này sẽ tự động cập nhật dữ liệu ở module khác mà không cần đồng bộ thủ công.

* **Luồng dữ liệu tự động hóa (Data Flow):** Trong kịch bản đặt hàng trực tuyến, luồng xử lý diễn ra như sau:

  * **Trigger (Kích hoạt):** Khách hàng hoàn tất việc thanh toán giỏ hàng trên Website Frontend.
  * **Write (Ghi dữ liệu):** Module eCommerce ghi nhận và tạo một Đơn bán hàng (Sales Order) vào Database.
  * **Sync (Đồng bộ ngầm):** Hệ thống lập tức gọi module Inventory (Kho bãi) để trừ đi số lượng tồn kho vật lý của mặt hàng tương ứng, ngăn chặn tình trạng bán vượt quá số lượng thực tế (Overselling).
  * **Report (Báo cáo):** Dữ liệu đơn hàng lập tức được tổng hợp và hiển thị trực quan trên Dashboard phân tích doanh thu của Quản trị viên.

### 2. Tích hợp Tự động hóa Đa dịch vụ (Cross-service Automation)

Để tối ưu hóa quy trình vận hành, Service 2 (Odoo) được thiết kế để liên kết trực tiếp với Service 1 (File Automation Engine) tạo thành một hệ sinh thái cảnh báo và bảo vệ dữ liệu khép kín:

* **Cảnh báo "Đơn hàng" và "Hết hàng" qua Telegram:** Tích hợp cơ chế Webhook/API để khi có đơn hàng mới hoặc khi sản phẩm trong kho giảm xuống mức 0, Odoo sẽ phát tín hiệu sang Node-RED. Node-RED xử lý và lập tức gửi tin nhắn thông báo đến telegram của quản trị viên.
* **Automated Database Backup (Sao lưu cơ sở dữ liệu ngầm):** Odoo được cấu hình để tự động xuất dữ liệu (Database Dump) định kỳ. Tệp tin sao lưu này được đẩy vào vùng `shared_data`. Ngay lập tức, luồng Radar của Service 1 sẽ phát hiện, bóc tách file và áp dụng luồng Shadow Backup để cất giữ vào vùng an toàn, đồng thời báo cáo trạng thái sao lưu qua Telegram.
