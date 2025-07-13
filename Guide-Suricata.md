# Hướng dẫn sử dụng rule Suricata
## Cấu trúc tổng thể của một Suricata rule
__<action> <protocol> <src_ip> <src_port> -> <dst_ip> <dst_port> ( <rule_options> )__  
  
Ví dụ tổng quát: alert tcp $HOME_NET any -> $EXTERNAL_NET 80 (msg:"Potential malware EXE download"; flow:to_client, established; content:"MZ"; depth:2; file_data; classtype:trojan-activity; sid:1000010; rev:1;)  

## Thành phần của một rule
__alert__: Hành động (có thể là alert, drop, reject, pass, rejectsrc, rejectdst)  
__tcp__: Giao thức (tcp, udp, icmp, ip, ...)  
__$HOME_NET__: Biến đại diện mạng nội bộ (khai báo trong suricata.yaml)
__any__: 	Cổng nguồn hoặc đích (có thể là số cụ thể hoặc any)
__->__ : Chiều của luồng (client → server hoặc ngược lại <-)
__( ... )__ : Phần rule options  

## Phần rule options: các option chính
__msg:__ Nội dung cảnh báo hiển thị  
__flow:__ 	Chỉ định hướng lưu lượng (to_server, to_client, established, not_established)  
__content:__ 	Tìm kiếm chuỗi trong payload  
__file_data;__ 	Bắt đầu kiểm tra nội dung file (trong HTTP, SMTP, FTP…)  
__http_uri;__, __http_host;__, __http_user_agent;__ Kiểm tra thành phần HTTP  
__pcre:__ Biểu thức chính quy  
__classtype:__ Phân loại cảnh báo (trojan-activity, web-application-attack…)  
__sid:__ 	Mã số định danh rule (unique)  
__rev:__ Số version của rule  

## Một số option quan trọng khác
__threshold:__ 	Giảm cảnh báo trùng (ví dụ 5 lần trong 60s mới alert)  
__flowbits:__ Theo dõi trạng thái giữa các rule  
__tag:__ 	Gắn tag để theo dõi lưu lượng sau khi rule match  
__dsize:__ 	Kiểm tra độ dài payload  
__byte_test:__ / byte_jump:__ Phân tích nhị phân  
__metadata:__ 	Thêm mô tả không ảnh hưởng logic rule  



