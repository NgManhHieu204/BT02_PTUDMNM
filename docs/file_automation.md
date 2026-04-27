# Cấu trúc thư mục triển khai (Directory Structure)

Hệ thống được tổ chức theo kiến trúc container hóa, trong đó các service được định nghĩa thông qua Docker Compose. Dữ liệu được phân mảnh và cô lập thông qua cơ chế Volume để đảm bảo tính bền vững (Persistent), khả năng mở rộng và an toàn thông tin.

```text
file-automation/
├── docker-compose.yml   # Định nghĩa các service và kiến trúc hệ thống (Docker Compose)
├── .gitignore
├── fb_data/             # Phân vùng CSDL - Chứa tệp tin filebrowser.db và các tệp tin transaction của SQLite
├── shared_data/         # Phân vùng thao tác (Workspace) - Thư mục dùng chung giữa các service
├── backup_data/         # Phân vùng an toàn (Vault) - Lưu trữ các bản sao lưu ngầm (Shadow Backup), cách ly với End-user
├── nodered_data/        # Dữ liệu persistent của Node-RED (flows, node modules, cấu hình)
└── node-red-custom/
    └── Dockerfile       # Chỉ thị đóng gói (Dockerfile) dùng để build custom Node-RED image
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

## Tạo Bot và lấy Token

Node-RED chỉ là một máy chạy ngầm, nó cần một con Bot Telegram để gửi tin báo động về điện thoại. Cần lấy 2 thông số cực kỳ quan trọng: Bot Token và Chat ID.

- Tải và cài đặt telegram sau đó mở ứng dụng telegram lên
- Vào ô tìm kiếm gõ: @BotFather (chọn cái có tích xanh)

  <img width="989" height="764" alt="image" src="https://github.com/user-attachments/assets/fd336f87-e641-4253-b37f-b85b6770e262" />

- Bấm Start, sau đó gõ lệnh: /newbot
- BotFather sẽ hỏi tên hiển thị của Bot, nhập tên bot. Ví dụ: `My Server Admin Bot`
- Tiếp theo, nhập username cho Bot (bắt buộc phải viết liền và kết thúc bằng chữ bot). Ví dụ: `hieu_server_bot`
- Nếu thành công, BotFather sẽ trả về một tin nhắn dài. Đó là đoạn mã HTTP API Token

  <img width="1919" height="1079" alt="Screenshot 2026-04-26 151136" src="https://github.com/user-attachments/assets/8ed1610a-1edb-4817-8f01-fc7f60f62260" />

## Kích hoạt Bot và lấy Chat ID cá nhân

Token là để Node-RED điều khiển con Bot còn Chat ID là để con Bot biết phải gửi tin nhắn cho ai.

- Bấm vào link username của con Bot vừa tạo (ví dụ @hieu_server_bot) để mở khung chat với nó
- Bấm Start và gửi cho nó một tin nhắn bất kỳ để mở khóa kênh giao tiếp

  <img width="1062" height="824" alt="image" src="https://github.com/user-attachments/assets/02f9903a-6549-4be1-8720-4b79bf30c6d3" />

- Tiếp theo, quay lại ô tìm kiếm của Telegram, tìm con Bot có tên là: @userinfobot (hoặc @RawDataBot)
- Bấm Start với con bot này và nó sẽ lập tức trả về thông tin cá nhân, nhìn vào dòng Id đây chính là Chat ID.

  <img width="1060" height="822" alt="image" src="https://github.com/user-attachments/assets/b4cb35ce-1131-4546-9b0b-8e7d7d825d13" />

# Tính năng Shadow Backup & Cảnh báo Telegram

## Khai báo Telegram Bot với Node-RED

- Ở cột Menu bên trái, tìm nhóm telegram, kéo node có tên là telegram sender ra màn hình
- Nhấp đúp chuột vào node vừa kéo ra. Ở mục Bot, bấm vào Add new telegram bot
  
  <img width="1913" height="1015" alt="image" src="https://github.com/user-attachments/assets/76a641f8-9ff0-432f-b96e-769a7ab7fb58" />

- Điền Bot-Name và dán chuỗi Token vừa lấy được vào ô Token. Bấm Add (Thêm), sau đó bấm Done (Xong)

  <img width="979" height="415" alt="image" src="https://github.com/user-attachments/assets/665be8fa-d6fa-4d73-9626-42f81e13cde5" />

## Giám sát thư mục File Browser

- Ở cột Menu trái cuộn lên nhóm storage kéo node watch directory ra màn hình
- Nhấp đúp vào node đó để cấu hình node watch directory số 1 làm sự kiện Upload
  
  <img width="735" height="532" alt="image" src="https://github.com/user-attachments/assets/5c4f6347-e3cc-4955-a536-051e5be6f91f" />

- Cấu hình node watch directory số 2 làm sự kiện Modify
  
  <img width="742" height="568" alt="image" src="https://github.com/user-attachments/assets/f36f287a-1d6a-4094-8ee1-3cbd1ac422d4" />

- Cấu hình node watch directory số 3 làm sự kiện xóa file

  <img width="712" height="536" alt="image" src="https://github.com/user-attachments/assets/1d15b97b-f601-4745-a54a-dfca6143bb58" />

## Test nhanh

### Debug Node

- Ở cột menu trái, tìm node debug và kéo ra màn hình

- Nối Rd1 (Sự kiện Upload) vào đầu cái node debug

- Bấm nút Deploy (Triển khai) để lưu lại hệ thống

  <img width="1546" height="726" alt="image" src="https://github.com/user-attachments/assets/1e7240c9-4992-4847-9fc7-6303676b3df8" />

### Kích hoạt

- Mở một tab trình duyệt rồi vào File Browser, upload thử một file bất kỳ

  <img width="1841" height="779" alt="image" src="https://github.com/user-attachments/assets/2f3f4dae-ff93-42cf-bf94-3f1823e4d3fb" />

- Quay lại màn hình Node-RED, ở bên phải màn hình, bấm vào cái biểu tượng con bọ (bug) để mở cửa sổ Debug Messages

  <img width="434" height="280" alt="image" src="https://github.com/user-attachments/assets/7f75af1e-a562-4116-8051-de8f5b9f7ad8" />

- Nhìn vào cửa sổ Debug, chúng ta có thể thấy SK1 không trả về một cục dữ liệu phức tạp nào mà nó chỉ đưa ra đúng một chuỗi ký tự (string) là đường dẫn tuyệt đối của cái file vừa được tạo: `msg.payload = "/shared_data/filetext"`

## Gửi tin nhắn về telegram khi có file Upload

Telegram Bot không hiểu đường dẫn file là gì nên phải gói tin nhắn thành một cấu trúc chuẩn (có Chat ID, có nội dung chữ).

- Ở cột menu trái, tìm nhóm function rồi kéo một node có tên là function ra màn hình
- Đặt vào giữa node Sự kiện Upload và node Telegram sender
- Nhấp đúp vào node function để thêm code rồi đổi tên thành: Tạo tin nhắn Upload

  <img width="1048" height="668" alt="image" src="https://github.com/user-attachments/assets/395842ac-5867-44f4-8ca6-a7f982b8ad10" />
  
- Nối dây Sự kiện Upload ➔ Tạo tin nhắn Upload ➔ Telegram sender xong bấm Deploy để triển khai

  <img width="1893" height="978" alt="image" src="https://github.com/user-attachments/assets/06881a79-1b08-458e-92a7-8250d8e15132" />

### Lỗi phát sinh khi deoploy: Node-RED không thể gửi tin nhắn Telegram (Lỗi `AggregateError`)

- Khi Upload 1 file pdf lên File Browser mặc dù Node-RED bắt được dữ liệu nhưng báo lỗi tại node Telegram Sender:
`Error: SLIGHTLYBETTEREFATAL: AggregateError`
Trạng thái của node Telegram không hiện chữ `connected` mà báo lỗi đỏ, tin nhắn không được gửi về điện thoại.

  <img width="1842" height="967" alt="image" src="https://github.com/user-attachments/assets/7e1cba5f-23c8-4c60-bca3-74c2063d269e" />

### Phân tích nguyên nhân cốt lõi

- Lỗi này không phải do cấu hình luồng Node-RED sai hay Token không hợp lệ, mà là một vấn đề liên quan đến cơ chế mạng của **Node.js** (nền tảng cốt lõi chạy Node-RED) bên trong Docker container.
- Kể từ phiên bản **Node.js 17** trở lên thì cơ chế phân giải tên miền (DNS Resolution) đã bị thay đổi. Mặc định, Node.js sẽ ưu tiên phân giải và kết nối thông qua địa chỉ mạng **IPv6** trước khi thử **IPv4**.
- Tuy nhiên, trong hệ thống mạng của Docker hoặc máy chủ vật lý (Homelab) trong bài của em chưa được cấu hình định tuyến IPv6 ra ngoài Internet một cách hoàn chỉnh. Khi Node-RED cố gắng gọi API của Telegram (`api.telegram.org`) bằng IPv6, kết nối bị kẹt, dẫn đến lỗi timeout và văng ra mã lỗi `AggregateError` (lỗi tổng hợp do nhiều nỗ lực kết nối đều thất bại).

### Giải pháp khắc phục

- Cần can thiệp vào biến môi trường (Environment Variable) của container Node-RED để ép nền tảng Node.js quay trở lại cách cũ: **Ưu tiên sử dụng IPv4**.

- Thêm cờ cấu hình `NODE_OPTIONS=--dns-result-order=ipv4first` vào file `docker-compose.yml`:

<img width="701" height="427" alt="image" src="https://github.com/user-attachments/assets/9ed6e397-91ff-4f15-95f3-0235f31bc18d" />

- Biến NODE_OPTIONS cho phép truyền các tham số khởi động trực tiếp vào runtime của Node.js. Khi sử dụng cờ --dns-result-order=ipv4first, hệ thống sẽ ưu tiên phân giải và kết nối tới máy chủ Telegram thông qua địa chỉ IPv4, thay vì thử IPv6 trước. Cách cấu hình này giúp tránh các vấn đề liên quan đến IPv6, từ đó đảm bảo kết nối ổn định và việc gửi tin nhắn diễn ra thông suốt.

### Kiểm thử

- Tại trang File Browser upload một file PDF

<img width="1902" height="1023" alt="image" src="https://github.com/user-attachments/assets/1de3716e-0fd9-406a-afd9-162861fcafbc" />

- Kết quả:

  <img width="1918" height="1014" alt="image" src="https://github.com/user-attachments/assets/c23ed20a-762b-4a96-b95a-2156832dc8df" />

## Xây dựng Shadow Backup Flow (luồng sao lưu tự động)

### Cấu hình Node Function

- Kéo một node function mới ra màn hình và đặt nằm dưới node Tạo tin nhắn Upload

- Nhấp đúp vào node, thêm code và đổi tên node thành: Lệnh Backup

  <img width="856" height="568" alt="image" src="https://github.com/user-attachments/assets/a06e6075-e618-4b0b-ae85-7ae960d9668d" />

### Cấu hình Node Exec

Node-RED đang chạy trong Docker nên sẽ dùng node exec để ra lệnh cho Linux tự động copy file, tốc độ sẽ rất nhanh.

- Ở cột menu trái, tìm node có tên là exec, kéo nó ra màn hình và đặt đằng sau node Lệnh Backup

- Nhấp đúp vào node exec, để trống ô Command hoàn toàn (vì node sẽ tự lấy lệnh cp từ hàm function ở trên truyền sang)

  <img width="707" height="617" alt="image" src="https://github.com/user-attachments/assets/81c19aed-16ea-47aa-95b2-3517c1ec2338" />

- Nối dây từ node Sự kiện Upload ➔ Lệnh Backup ➔ node exec

  <img width="1911" height="866" alt="image" src="https://github.com/user-attachments/assets/84ea4591-00db-4ee9-b234-5b4005bef7fa" />

### Kiểm thử

- Upload 1 file bất kì lên File Browser

  <img width="1902" height="913" alt="image" src="https://github.com/user-attachments/assets/01b69621-58be-409f-8aff-ffa347eb5eaa" />

- Gõ lệnh `ls -l ~/my-homelab-project/file-automation/backup_data` để xem kết quả

  <img width="878" height="106" alt="image" src="https://github.com/user-attachments/assets/b629dde0-d9f2-4e18-a1e1-7e63c7ff13c9" />

# Xử lý sự kiện Modify và Cơ chế Versioning

Nếu luồng `Upload` hoặc luồng `Modify` cứ copy một file có cùng tên (ví dụ: bao_cao.pdf), hệ điều hành sẽ tự động ghi đè (overwrite) lên bản backup cũ. Hậu quả sẽ làm mất lịch sử chỉnh sửa. Để giải vấn đề này, em sẽ áp dụng cơ chế File Versioning (Đánh dấu phiên bản theo thời gian)

## Sự kiện Modify

- Kéo một node function mới ra màn hình và đặt ngay sau node Sự kiện Modify

- Đổi tên node thành: Xử lý Versioning & Backup và thêm code vào (sử dụng thuật toán bóc tách phần mở rộng của file, sau đó chèn thêm chuỗi thời gian thực YYYYMMDD_HHMMSS vào giữa để đảm bảo tên file backup luôn là duy nhất)

  <img width="1121" height="882" alt="image" src="https://github.com/user-attachments/assets/8a6e8f44-bcf8-41e2-ad95-23becfdf0fca" />

- Nối node Sự kiện Modify ➔ Xử lý Versioning & Backup ➔ node exec

  <img width="1586" height="851" alt="image" src="https://github.com/user-attachments/assets/6634d580-b78d-41cb-8d91-1f2d0c94a20b" />

## Thiết lập tiến trình Cảnh báo

- Kéo một node function mới ra màn hình và đổi tên thành: Tin nhắn Cảnh báo Modify, thêm code để để định dạng gói tin (Payload) giao tiếp với API Telegram

  <img width="993" height="603" alt="image" src="https://github.com/user-attachments/assets/9dddf9d6-1e22-4e45-9ead-9913e428313f" />

- Nối node Sự kiện Modify ➔ Tin nhắn Cảnh báo Modify ➔ Telegram sender và bấm Deploy để triển khai

  <img width="1874" height="797" alt="image" src="https://github.com/user-attachments/assets/55ee89bd-7416-4580-a74e-e94ded3a9086" />

## Kiểm thử

- Tại File Browser, ở 1 file text ban đầu upload (không có nội dung bên trong) thêm nội dung và bấm Save để lưu lại và ngay lập tức Telegram nhận được tin nhắn cảnh báo bảo mật kèm tên file

  <img width="1915" height="919" alt="image" src="https://github.com/user-attachments/assets/a448d89a-0f97-4e9d-83f7-c63e01f0cd66" />

- Chạy lệnh `ls -l ~/my-homelab-project/file-automation/backup_data` để xem phiên bản mới của tệp tin phải được lưu thành công vào phân vùng an toàn mà không làm ảnh hưởng đến bản gốc, hơn nữa khi bị chỉnh sửa nhiều lần, hệ thống cũng sao lưu ra nhiều bản (không bị ghi đè lên bản sao lưu cũ)

  <img width="966" height="154" alt="image" src="https://github.com/user-attachments/assets/9d9e7d17-412e-4c9f-82db-d55a6ee31bea" />

  <img width="967" height="169" alt="image" src="https://github.com/user-attachments/assets/50acdb49-2fc9-47cd-af94-22d7db967082" />

# Triển khai luồng sự kiện Delete

Khác với sự kiện Upload hoặc Modify, khi sự kiện Delete xảy ra trên File Browser, tệp tin vật lý trong `shared_data` đã bị tiêu hủy. Tuy nhiên, nhờ cơ chế Shadow Backup đã xây dựng trước đó, bản gốc và các bản sửa đổi đều đã nằm an toàn trong `backup_data`. Do đó, đối với sự kiện này không cần gọi lệnh thực thi sao lưu (Exec node) nữa mà chỉ cần phát luồng Cảnh báo.

## Thiết lập tiến trình Cảnh báo

- Kéo một node `function` mới ra màn hình và đặt sau node `Sự kiện Delete`
- Đổi tên node thành: `Cảnh báo Delete` thêm code để cấu trúc gói tin cảnh báo

  <img width="962" height="709" alt="image" src="https://github.com/user-attachments/assets/bbb2d71f-ef9a-4016-9e66-20d1bf4124e0" />

- Nối node Sự kiện Delete ➔ Tin nhắn Cảnh báo Delete ➔ Telegram sender

  <img width="1322" height="777" alt="image" src="https://github.com/user-attachments/assets/4b10610d-4ab8-4908-a33f-343762b0117c" />

## Kiểm thử

- Truy cập vào File Browser, chọn một tệp tin bất kỳ và thực hiện lệnh Xóa (Delete)

  <img width="1879" height="982" alt="image" src="https://github.com/user-attachments/assets/05d8a691-d3b8-45b6-ae93-37ae72119243" />

- Ngay sau khi thực hiện lệnh Xóa, có tin nhắn gửi về telegram ngay lập tức

  <img width="1522" height="807" alt="image" src="https://github.com/user-attachments/assets/1ebc0af8-24e1-42a7-99a2-92f0660d919b" />

- Chạy lệnh `ls -l ~/my-homelab-project/file-automation/backup_data` để kiểm tra trong thư mục backup_data bản sao lưu của tệp tin này vẫn phải được bảo toàn nguyên vẹn
  
  <img width="1013" height="193" alt="image" src="https://github.com/user-attachments/assets/802d6342-5c52-4622-9397-105436d2026c" />

- Khôi phục file: Xóa 1 file text trên File Browser 

  <img width="1462" height="757" alt="image" src="https://github.com/user-attachments/assets/4d5c769e-4fb4-4c13-a07f-9d48dad3b799" />

- Chạy lệnh `cp ~/my-homelab-project/file-automation/backup_data/filetext_modified_20260426_153318 ~/my-homelab-project/file-automation/shared_data/filetext` để đưa bản sao lưu từ phân vùng an toàn ngược trở lại phân vùng thao tác của End-user, đồng thời đổi tên file về nguyên bản
  
  <img width="1464" height="86" alt="image" src="https://github.com/user-attachments/assets/a97b7a7d-be0a-4b63-b059-342986fa49d2" />

- Kiểm tra lại trên File Browser

  <img width="1534" height="622" alt="image" src="https://github.com/user-attachments/assets/7d0ca03c-1c97-4436-8f09-0d6278cf5ad2" />

# Đánh giá rủi ro và Hạn chế đã biết

Trong quá trình Kiểm thử hệ thống em đã xác định một hạn chế liên quan đến luồng sự kiện thư mục (Directory Events)

## Mô tả

Hệ thống Node-RED được cấu hình với tham số `Depth = 0` và tập lệnh sao lưu `cp` không sử dụng cờ đệ quy (`-r`). Do đó, hệ thống chỉ hỗ trợ giám sát và Backup tự động đối với các **tệp tin đơn lẻ (Single Files)** ở phân vùng gốc.

## Hành vi hệ thống 

Khi người dùng tải lên toàn bộ một thư mục (Folder), hệ thống sẽ không phát sinh cảnh báo và không sao lưu ngầm nội dung bên trong thư mục đó.

  <img width="1465" height="812" alt="image" src="https://github.com/user-attachments/assets/05c9d56f-e07f-47cb-8a84-9b32547bfaba" />

## Biện pháp quản lý (Workaround)

- Để đảm bảo tính toàn vẹn dữ liệu và tránh tình trạng nghẽn cổ chai (Spam API) khi tải lên hàng ngàn tệp tin cùng lúc, quy trình vận hành tiêu chuẩn yêu cầu người dùng **phải nén thư mục thành định dạng Archive (.zip, .rar, .tar.gz)** trước khi tải lên File Browser. Hệ thống sẽ nhận diện và sao lưu tệp tin nén này một cách hoàn hảo.

- Upload 1 file .zip lên File Browser
  
  <img width="1485" height="901" alt="image" src="https://github.com/user-attachments/assets/2d01a3f6-a460-4926-b51a-9e29504880e5" />

- Kiểm tra trong thư mục Backup_data

  <img width="973" height="310" alt="image" src="https://github.com/user-attachments/assets/c73ca58e-2984-4660-b3a7-96203b97c8cc" />

## Hướng phát triển (Future Work)

Trong các phiên bản sau, có thể nghiên cứu tích hợp thêm kịch bản bash shell tự động nén (auto-zip) trên server khi phát hiện sự kiện tải lên thư mục.

# Tối ưu hóa Triển khai với Docker Hub

- Đăng nhập Docker Hub bằng cách chạy lệnh `docker login`

  <img width="1234" height="285" alt="image" src="https://github.com/user-attachments/assets/54bd13f3-450e-493d-80e5-6126ffd0f2ce" />

- Tại Trang `https://login.docker.com/activate` nhập code và đăng nhập vào Docker Hub

  <img width="1531" height="869" alt="image" src="https://github.com/user-attachments/assets/0fa94060-67c5-4568-af03-d3de8fee0bea" />

- Đăng nhập thành công:

  <img width="1259" height="432" alt="image" src="https://github.com/user-attachments/assets/3e3584b4-7f5a-4b23-bba9-97d0cadbf1bf" />

- Đóng gói và Đẩy lên Docker Hub

  - Chạy lệnh `docker build -t mhieu2004/node-red-shadow-backup:v1 ./node-red-custom` để build images

    <img width="1466" height="465" alt="image" src="https://github.com/user-attachments/assets/e7bc7185-1c8a-4864-bcc8-f9e2d4e8989d" />

  - Chạy lệnh `docker push mhieu2004/node-red-shadow-backup:v1` để push lên Docker Hub
 
    <img width="1207" height="540" alt="image" src="https://github.com/user-attachments/assets/247a7192-96b5-4006-bb4d-b05a9ee1a35a" />

- Cập nhật docker-compose.yml: thay `build: ./node-red-custom` thành `image: mhieu2004/node-red-shadow-backup:v1`

  <img width="982" height="618" alt="image" src="https://github.com/user-attachments/assets/1f531413-445a-4e7c-970c-86d1d8cea3ab" />

- Lợi ích
  
  - Loại bỏ hoàn toàn rủi ro khi cài đặt thư viện (`node-red-contrib-watchdirectory`, `node-red-contrib-telegrambot`).
    
  - Thời gian triển khai giảm từ vài phút xuống còn vài giây.
    
  - Đảm bảo tính nhất quán (Consistency) 100% trên mọi môi trường máy chủ khác nhau (Dev/Test/Prod). Môi trường mới chỉ cần tải file `docker-compose.yml` và chạy `docker compose up -d`.
