LAB 01 -- An toàn bảo mật hệ thống thông tin

1. Thông tin bài Lab

Lab: Lab 01

Mô hình: Client + Attacker (Windows 11) ↔ Server (Ubuntu Server
24.04.3 LTS)

Mạng: VMware Host-only -- VMnet1 -- 192.168.56.0/24

2. Mục tiêu

Thiết lập môi trường Server/Client trên mạng Host-only.

Cấu hình và kiểm tra dịch vụ Telnet (TCP/23) và SSH
(TCP/22).

Dùng Wireshark bắt và phân tích gói tin.

Quan sát khả năng lộ thông tin của Telnet.

So sánh Telnet với SSH về Confidentiality, Integrity và
Authentication.

Thực hành đăng nhập SSH bằng public-key authentication.

Tìm hiểu một số biện pháp hardening cho SSH.

3. Môi trường thực hành

Thành phần          Cấu hình

Server              Ubuntu Server 24.04.3 LTS
Hostname            ubuntu-srv01
Server IP           192.168.56.10
Client + Attacker   Windows 11
Client IP           192.168.56.1
Virtualization      VMware Workstation Pro 17.6.3
Network             Host-only VMnet1
Packet Capture      Wireshark 4.6.8 + Npcap 1.89
SSH Client          PuTTY 0.85 + PuTTYgen
Telnet              inetutils-telnetd / openbsd-inetd
SSH                 OpenSSH 9.6p1

4. Quá trình thực hiện

Bước 1 -- Thiết lập Server

Tạo máy ảo Ubuntu Server trên VMware.

Chọn card mạng Host-only VMnet1.

Cấu hình Server với IP 192.168.56.10/24.

Kiểm tra kết nối giữa Server và Windows bằng ping.

Cài đặt dịch vụ Telnet.

Kiểm tra SSH đang lắng nghe trên TCP/22.

Bước 2 -- Bắt gói Telnet

Wireshark được cấu hình trên card VMware Network Adapter VMnet1.

Capture filter:

host 192.168.56.10 and (tcp port 23 or tcp port 22)

Sau đó dùng PuTTY kết nối Telnet tới:

192.168.56.10:23

Thực hiện các lệnh:

whoami
pwd
ls -la
mkdir lab1_test
cat /etc/os-release
exit

Bước 3 -- Phân tích Telnet với mật khẩu đơn giản

Kết quả từ telnet_capture.pcapng:

Bắt được 515 gói trong khoảng 391 giây.

Có 324 gói TELNET.

Có 222 gói chứa 1 byte dữ liệu.

Dùng Follow TCP Stream có thể đọc được username, password, lệnh
và kết quả lệnh.

Telnet truyền dữ liệu dạng rõ, không mã hóa.

Bước 4 -- Phân tích Telnet với mật khẩu phức tạp

Đổi mật khẩu sang mật khẩu dài và phức tạp rồi bắt gói lại.

Kết quả từ telnet_capture_2.pcapng:

Bắt được 211 gói trong khoảng 58 giây.

Username và mật khẩu mới vẫn xuất hiện dạng rõ trong TCP payload.

Mật khẩu phức tạp không khắc phục được vấn đề nghe lén của
Telnet.

Kết luận: điểm yếu nằm ở việc Telnet không mã hóa kênh truyền, không
phụ thuộc vào độ phức tạp của mật khẩu.

Bước 5 -- Bắt và phân tích SSH

Kết nối PuTTY tới:

192.168.56.10:22

Thực hiện các lệnh tương tự Telnet.

Kết quả:

Phiên SSH được mã hóa sau quá trình trao đổi khóa.

Wireshark chỉ hiển thị các gói Encrypted packet.

Không thể đọc username, password, lệnh và kết quả lệnh.

Vẫn quan sát được một số metadata như IP, port, thời gian và độ dài
gói.

Bước 6 -- Xác thực SSH bằng Public Key

Tạo cặp khóa Ed25519 bằng PuTTYgen.

Lưu private key thành lab1_key.ppk.

Đưa public key vào:

~/.ssh/authorized_keys

Cấu hình quyền:

.ssh → 700

authorized_keys → 600

Cấu hình PuTTY sử dụng private key.

Đăng nhập SSH thành công mà không cần nhập password.

5. Kết quả chính

Telnet

TCP port: 23

Không mã hóa.

Có thể đọc username/password.

Có thể đọc lệnh và kết quả lệnh.

Không có cơ chế xác thực máy chủ.

Mức độ bảo mật thấp.

SSH

TCP port: 22

Mã hóa phiên làm việc.

Không đọc được username/password và nội dung lệnh qua Wireshark.

Có host key để xác thực máy chủ.

Hỗ trợ password và public-key authentication.

Có cơ chế kiểm tra toàn vẹn.

6. So sánh nhanh

Tiêu chí            Telnet                        SSH

Port                TCP/23                        TCP/22
Mã hóa              Không                         Có
Username/Password   Đọc được                      Không đọc được
Lệnh và kết quả     Đọc được                      Không đọc được
Xác thực máy chủ    Không                         Có
Public Key          Không                         Có
Kiểm tra toàn vẹn   Không                         Có
Metadata            Có                            Có
Mục đích thực tế    Lab/mạng cô lập/thiết bị cũ   Quản trị từ xa

7. Kết luận

Qua thực nghiệm với Wireshark, Lab 01 cho thấy sự khác biệt rõ ràng giữa
Telnet và SSH:

Telnet truyền dữ liệu dạng rõ nên thông tin đăng nhập và nội
dung phiên có thể bị nghe lén.

Việc sử dụng mật khẩu dài và phức tạp không giải quyết được vấn đề
này.

SSH mã hóa nội dung phiên, đồng thời hỗ trợ xác thực máy chủ,
kiểm tra toàn vẹn và xác thực bằng public key.

Trong môi trường thực tế, nên sử dụng SSH thay cho Telnet và thực
hiện các biện pháp hardening như tắt password authentication khi phù
hợp, cấm root login, giới hạn user/nguồn truy cập và gỡ Telnet.

8. File thực nghiệm

Các file bắt gói được sử dụng trong Lab:

telnet_capture.pcapng
telnet_capture_2.pcapng
telnet_ssh_capture.pcapng

telnet_capture.pcapng: phiên Telnet với mật khẩu đơn giản.

telnet_capture_2.pcapng: phiên Telnet với mật khẩu phức tạp.

telnet_ssh_capture.pcapng: phiên bắt chung Telnet và SSH.

9. Tài liệu tham khảo / Minh chứng

Báo cáo thực hành Lab 01.

Wireshark packet capture.

PuTTY / PuTTYgen.

Ubuntu Server 24.04.3 LTS.

Video thực hành: https://youtu.be/TCti74hNbcQ
