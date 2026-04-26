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

- Truy cập địa chỉ `192.168.153.128:1880` sẽ thấy giao diện của Node-RED

  <img width="1908" height="1016" alt="image" src="https://github.com/user-attachments/assets/d23a1f40-8856-4854-ba61-4d0b7dd3298d" />

- Truy cập địa chỉ `192.168.153.128:8080` sẽ thấy giao diện của file browser, tài khoản mặc định là admin và mật khẩu là 1 chuỗi ngẫu nhiên sinh ra,
xem bằng cách chạy lệnh `docker logs filebrowser_app`

  <img width="1158" height="310" alt="image" src="https://github.com/user-attachments/assets/ff03d7d7-0fe7-455b-bfe1-31bf4d3e0f2a" />

- Giao diện đăng nhập File Browser

  <img width="1877" height="1018" alt="image" src="https://github.com/user-attachments/assets/e2d36495-6799-4253-bfbb-d57214bc79fa" />

- Sau khi đăng nhập thành công thì nên đổi mật khẩu

  <img width="1917" height="1022" alt="image" src="https://github.com/user-attachments/assets/b82d627d-d955-4a6f-9196-947db6ff0363" />

# Tích hợp Shadow Backup & Audit Trail

- Di chuyển vào thư mục dự án và tạo thêm một thư mục backup

```text
cd ~/my-homelab-project/service-1-file-automation
mkdir backup_data
sudo chmod 777 backup_data
```
  <img width="970" height="124" alt="image" src="https://github.com/user-attachments/assets/017df693-b399-4b95-8720-4d05211d7af3" />

- Cập nhật file docker-compose.yml: Mở file bằng lệnh `nano docker-compose.yml` và thêm 1 dòng ở phần volumes của Node-RED

```text
volumes:
      - ./nodered_data:/data
      - ./shared_data:/shared_data
      - ./backup_data:/backup # THÊM DÒNG NÀY
```
 <img width="883" height="707" alt="image" src="https://github.com/user-attachments/assets/62a59c88-0189-4d68-92d6-947da1c687a0" />

- Khởi động lại bằng lệnh `docker compose up -d`

 <img width="1453" height="125" alt="image" src="https://github.com/user-attachments/assets/04b63bb1-d62b-48ac-bb7c-c2ac2288993c" />

# Khởi tạo Telegram Bot và Lấy thông tin

## Bước 1: Tạo Bot và lấy Token

Node-RED chỉ là một máy chạy ngầm, nó cần một con Bot Telegram để gửi tin báo động về điện thoại. Cần lấy 2 thông số cực kỳ quan trọng: Bot Token và Chat ID.

- Tải và cài đặt telegram sau đó mở ứng dụng telegram lên
- Vào ô tìm kiếm gõ: @BotFather (chọn cái có tích xanh)

  <img width="989" height="764" alt="image" src="https://github.com/user-attachments/assets/fd336f87-e641-4253-b37f-b85b6770e262" />

- Bấm Start, sau đó gõ lệnh: /newbot
- BotFather sẽ hỏi tên hiển thị của Bot, nhập tên bot. Ví dụ: `My Server Admin Bot`
- Tiếp theo, nhập username cho Bot (bắt buộc phải viết liền và kết thúc bằng chữ bot). Ví dụ: `hieu_server_bot`
- Nếu thành công, BotFather sẽ trả về một tin nhắn dài. Đó là đoạn mã HTTP API Token

  <img width="1919" height="1079" alt="Screenshot 2026-04-26 151136" src="https://github.com/user-attachments/assets/8ed1610a-1edb-4817-8f01-fc7f60f62260" />

## Bước 2: Kích hoạt Bot và lấy Chat ID cá nhân

Token là để Node-RED điều khiển con Bot còn Chat ID là để con Bot biết phải gửi tin nhắn cho ai.

- Bấm vào link username của con Bot vừa tạo (ví dụ @hieu_server_bot) để mở khung chat với nó
- Bấm Start và gửi cho nó một tin nhắn bất kỳ để mở khóa kênh giao tiếp

  <img width="1062" height="824" alt="image" src="https://github.com/user-attachments/assets/02f9903a-6549-4be1-8720-4b79bf30c6d3" />

- Tiếp theo, quay lại ô tìm kiếm của Telegram, tìm con Bot có tên là: @userinfobot (hoặc @RawDataBot)
- Bấm Start với con bot này và nó sẽ lập tức trả về thông tin cá nhân, nhìn vào dòng Id đây chính là Chat ID.

  <img width="1060" height="822" alt="image" src="https://github.com/user-attachments/assets/b4cb35ce-1131-4546-9b0b-8e7d7d825d13" />

# Tính năng Shadow Backup & Cảnh báo Telegram

## Bước 1: Khai báo Telegram Bot với Node-RED

- Ở cột Menu bên trái, tìm nhóm telegram, kéo node có tên là telegram sender ra màn hình
- Nhấp đúp chuột vào node vừa kéo ra. Ở mục Bot, bấm vào Add new telegram bot
  
  <img width="1913" height="1015" alt="image" src="https://github.com/user-attachments/assets/76a641f8-9ff0-432f-b96e-769a7ab7fb58" />

- Điền Bot-Name và dán chuỗi Token vừa lấy được vào ô Token. Bấm Add (Thêm), sau đó bấm Done (Xong)

  <img width="979" height="415" alt="image" src="https://github.com/user-attachments/assets/665be8fa-d6fa-4d73-9626-42f81e13cde5" />

## Bước 2: Giám sát thư mục File Browser

- Ở cột Menu trái cuộn lên nhóm storage kéo node watch directory ra màn hình
- Nhấp đúp vào node đó để cấu hình node watch directory số 1 làm sự kiện Upload
  
  <img width="735" height="532" alt="image" src="https://github.com/user-attachments/assets/5c4f6347-e3cc-4955-a536-051e5be6f91f" />

- Cấu hình node watch directory số 2 làm sự kiện sửa file
  
  <img width="742" height="568" alt="image" src="https://github.com/user-attachments/assets/f36f287a-1d6a-4094-8ee1-3cbd1ac422d4" />

- Cấu hình node watch directory số 3 làm sự kiện xóa file

  <img width="712" height="536" alt="image" src="https://github.com/user-attachments/assets/1d15b97b-f601-4745-a54a-dfca6143bb58" />

## Test nhanh

### B1: Debug Node

- Ở cột menu trái, tìm node debug và kéo ra màn hình

- Nối Rd1 (Sự kiện Upload) vào đầu cái node debug

- Bấm nút Deploy (Triển khai) để lưu lại hệ thống

  <img width="1546" height="726" alt="image" src="https://github.com/user-attachments/assets/1e7240c9-4992-4847-9fc7-6303676b3df8" />

### B2: Kích hoạt

- Mở một tab trình duyệt rồi vào File Browser, upload thử một file bất kỳ

  <img width="1841" height="779" alt="image" src="https://github.com/user-attachments/assets/2f3f4dae-ff93-42cf-bf94-3f1823e4d3fb" />

- Quay lại màn hình Node-RED, ở bên phải màn hình, bấm vào cái biểu tượng con bọ (bug) để mở cửa sổ Debug Messages

  <img width="434" height="280" alt="image" src="https://github.com/user-attachments/assets/7f75af1e-a562-4116-8051-de8f5b9f7ad8" />

- Nhìn vào cửa sổ Debug, chúng ta có thể thấy Rd1 không trả về một cục dữ liệu phức tạp nào mà nó chỉ đưa ra đúng một chuỗi ký tự (string) là đường dẫn tuyệt đối của cái file vừa được tạo: `msg.payload = "/shared_data/filetext"`

# Gửi tin nhắn về telegram khi có file Upload

Telegram Bot không hiểu đường dẫn file là gì nên phải gói tin nhắn thành một cấu trúc chuẩn (có Chat ID, có nội dung chữ).

- Ở cột menu trái, tìm nhóm function rồi kéo một node có tên là function ra màn hình
- Đặt vào giữa node Sự kiện Upload và node Telegram sender
- Nhấp đúp vào node function để thêm code rồi đổi tên thành: Tạo tin nhắn Upload

  <img width="1048" height="668" alt="image" src="https://github.com/user-attachments/assets/395842ac-5867-44f4-8ca6-a7f982b8ad10" />
  
- Nối dây Sự kiện Upload ➔ Tạo tin nhắn Upload ➔ Telegram sender xong bấm Deploy để triển khai

  <img width="1893" height="978" alt="image" src="https://github.com/user-attachments/assets/06881a79-1b08-458e-92a7-8250d8e15132" />

## Lỗi phát sinh khi deoploy: Node-RED không thể gửi tin nhắn Telegram (Lỗi `AggregateError`)

- Khi Upload 1 file pdf lên File Browser mặc dù Node-RED bắt được dữ liệu nhưng báo lỗi tại node Telegram Sender:
`Error: SLIGHTLYBETTEREFATAL: AggregateError`
Trạng thái của node Telegram không hiện chữ `connected` mà báo lỗi đỏ, tin nhắn không được gửi về điện thoại.

  <img width="1842" height="967" alt="image" src="https://github.com/user-attachments/assets/7e1cba5f-23c8-4c60-bca3-74c2063d269e" />

## Phân tích nguyên nhân cốt lõi:

- Lỗi này không phải do cấu hình luồng Node-RED sai hay Token không hợp lệ, mà là một vấn đề liên quan đến cơ chế mạng của **Node.js** (nền tảng cốt lõi chạy Node-RED) bên trong Docker container.
- Kể từ phiên bản **Node.js 17** trở lên thì cơ chế phân giải tên miền (DNS Resolution) đã bị thay đổi. Mặc định, Node.js sẽ ưu tiên phân giải và kết nối thông qua địa chỉ mạng **IPv6** trước khi thử **IPv4**.
- Tuy nhiên, trong hệ thống mạng của Docker hoặc máy chủ vật lý (Homelab) trong bài của em chưa được cấu hình định tuyến IPv6 ra ngoài Internet một cách hoàn chỉnh. Khi Node-RED cố gắng gọi API của Telegram (`api.telegram.org`) bằng IPv6, kết nối bị kẹt, dẫn đến lỗi timeout và văng ra mã lỗi `AggregateError` (lỗi tổng hợp do nhiều nỗ lực kết nối đều thất bại).

## Giải pháp khắc phục

- Cần can thiệp vào biến môi trường (Environment Variable) của container Node-RED để ép nền tảng Node.js quay trở lại cách cũ: **Ưu tiên sử dụng IPv4**.

- Thêm cờ cấu hình `NODE_OPTIONS=--dns-result-order=ipv4first` vào file `docker-compose.yml`:

<img width="701" height="427" alt="image" src="https://github.com/user-attachments/assets/9ed6e397-91ff-4f15-95f3-0235f31bc18d" />

- Biến NODE_OPTIONS cho phép truyền các tham số khởi động trực tiếp vào runtime của Node.js. Khi sử dụng cờ --dns-result-order=ipv4first, hệ thống sẽ ưu tiên phân giải và kết nối tới máy chủ Telegram thông qua địa chỉ IPv4, thay vì thử IPv6 trước. Cách cấu hình này giúp tránh các vấn đề liên quan đến IPv6, từ đó đảm bảo kết nối ổn định và việc gửi tin nhắn diễn ra thông suốt.

## Kiểm thử

- Tại trang File Browser upload một file PDF

<img width="1902" height="1023" alt="image" src="https://github.com/user-attachments/assets/1de3716e-0fd9-406a-afd9-162861fcafbc" />

- Kết quả:

  <img width="1918" height="1014" alt="image" src="https://github.com/user-attachments/assets/c23ed20a-762b-4a96-b95a-2156832dc8df" />
