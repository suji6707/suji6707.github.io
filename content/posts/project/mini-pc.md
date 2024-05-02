+++
title = 'Mini-PC Ubuntu'
date = 2024-05-02T00:20:22+09:00
draft = true
+++
## 우분투 원격 서버 직접 설치하기

*****
UEFI stands for Unified Extensible Firmware Interface
- it stores all data about initialization and startup in an .efi file, instead of storing it on the firmware.
- This .efi file is stored on a special partition called EFI System Partition (ESP) on the hard disk. This ESP partition also contains the bootloader.

Rufus는 ISO 운영체제 파일이 설치되어있지 않은 컴퓨터에 USB로 설치하기 위해 필요.

Rufus는 USB 디스크에 운영 체제를 설치할 수 있도록 만드는 도구입니다. 이를 통해 사용자는 ISO 파일 형식의 운영 체제 이미지를 USB 드라이브에 쓰고(저장하고), 그 USB 드라이브를 사용하여 컴퓨터를 부팅하고 운영 체제를 설치할 수 있습니다
1. ISO 이미지(운영체제)를 선택하고 (다운받은)
2. USB 드라이브 포맷: Rufus는 선택된 USB 드라이브를 포맷하여 부팅 가능한 상태로 만듭니다. 이는 기존 데이터를 모두 지우고 새로운 파일 시스템을 설정하는 과정을 포함합니다.
3. ISO 이미지를 USB에 쓰기: 포맷된 USB 드라이브에 ISO 파일의 내용을 복사
4. 부팅 순서 설정: 컴퓨터의 UEFI 또는 BIOS 설정을 변경하여 USB 드라이브가 우선적으로 부팅되도록 설정
5. 운영 체제 설치: USB 드라이브를 통해 컴퓨터를 부팅하면, USB에 저장된 운영 체제 설치 프로그램이 실행됩니다. 

---
생각한 과정
- m1 mac ubuntu booting disk
- Create a bootable USB stick with Rufus on Windows
- ubuntu dual booting 
.. 그러나 우분투 server 패키지를 깔았다가 윈도우를 전부 밀어버렸다.
512GB를 전부 우분투에서 쓰게됨.

1. ssh 설정
2. 포트포워딩

1.
Ubuntu machine 와의 연결에 ssh 를 사용하기 위해서는 해당 Ubuntu machine에는 SSH Server가 설치되어 있어야 하고, 해당 Server에 접속하려는 PC 에서는 SSH Client 설치가 필요하다. 

what is my ip
->> 외부 퍼블릭 IP로 접속할 수 있도록

비밀번호 로그인 끄고 
반드시 pem 파일 로그인으로 ㄱ.

- 퍼블릭 키는 원격서버가 갖고있고, 
프라이빗키는 유저/클라이언트에게 줘야.
깃허브는 클라에서 생성해서 복호화 키를 깃헙 서버로 보내는거긴 한데. 그래서 둘 다 상관없음

암호화를 하고 복호화를 하는데
서버는 복호화 키가 있어야하고 클라에게 암호화키를 줘야
클라에서 암호화 키를 갖고 접속할 수 있음.

/home/jisu/.ssh/id_rsa -> my-server 로 .pem키 저장되었고
empty 비번. 다른사람이 접근할 수 없다는 전제하.


Upload the public certificate to to server
-> 내 경우 upload private to client

sudo update 
build essential
install vim

💎 authorized_keys 생성 필수임 **
-> vim으로 생성해서 pub 붙여넣기.

sudo ufw allow ssh, http
status 
-> 동일하게 나옴.

💎 원격서버의 유저, 비번으로 해야함!

## 포트포워딩


## disable ssh password login ubuntu
-> 비밀번호를 통한 로그인을 막기 위해
pem 키 없이 ssh 로 접근하려 했을때 

💎 etc/ssh/config/sshd_config.d
passwordAuthentication no
UsePAM no


vscode 연결 설치하기.
-> 

