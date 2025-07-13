# Hướng dẫn sử dụng rule Suricata
## Cấu trúc tổng thể của một Suricata rule
<action> <protocol> <src_ip> <src_port> -> <dst_ip> <dst_port> ( <rule_options> )  
Ví dụ tổng quát: alert tcp $HOME_NET any -> $EXTERNAL_NET 80 (msg:"Potential malware EXE download"; flow:to_client, established; content:"MZ"; depth:2; file_data; classtype:trojan-activity; sid:1000010; rev:1;)  

## Thành phần của một rule
alert: Hành động (có thể là alert, drop, reject, pass, rejectsrc, rejectdst)  
tcp: Giao thức (tcp, udp, icmp, ip, ...)  
$HOME_NET: Biến đại diện mạng nội bộ (khai báo trong suricata.yaml)
any: 	Cổng nguồn hoặc đích (có thể là số cụ thể hoặc any)
-> : Chiều của luồng (client → server hoặc ngược lại <-)
( ... ) : Phần rule options  

## Phần rule options: các option chính
msg: Nội dung cảnh báo hiển thị  
flow: 	Chỉ định hướng lưu lượng (to_server, to_client, established, not_established)  
content: 	Tìm kiếm chuỗi trong payload  
file_data; 	Bắt đầu kiểm tra nội dung file (trong HTTP, SMTP, FTP…)  
http_uri;, http_host;, http_user_agent; Kiểm tra thành phần HTTP  
pcre: Biểu thức chính quy  
classtype: Phân loại cảnh báo (trojan-activity, web-application-attack…)  
sid: 	Mã số định danh rule (unique)  
rev: Số version của rule  

## Một số option quan trọng khác
threshold: 	Giảm cảnh báo trùng (ví dụ 5 lần trong 60s mới alert)  
flowbits: Theo dõi trạng thái giữa các rule  
tag: 	Gắn tag để theo dõi lưu lượng sau khi rule match  
dsize: 	Kiểm tra độ dài payload  
byte_test: / byte_jump: Phân tích nhị phân  
metadata: 	Thêm mô tả không ảnh hưởng logic rule  



