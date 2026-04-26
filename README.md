# MÔN HỌC: PHÁT TRIỂN ỨNG DỤNG MÃ NGUỒN MỞ

# HỌ VÀ TÊN: NGUYỄN MẠNH HIẾU - MSSV: K225480106020 - LỚP: K58KTP

# Tổng quan

Dự án này là một hệ thống máy chủ cá nhân (Homelab) được xây dựng theo kiến trúc **Microservices phân tách độc lập**. Thay vì gom tất cả vào một cụm duy nhất, hệ thống được chia nhỏ thành các `services` riêng biệt.

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
