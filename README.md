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
