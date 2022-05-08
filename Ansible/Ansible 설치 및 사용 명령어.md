


## Ansible 구조
inventory
playbook
module

## 설치
 1) ansible 설치
 # epel 레포지토리 추가
 yum install -y epel-release 
 
 # epel 레포지토리 확인
 yum repolist

 # ansible 설치
  yum install -y ansible

 # ansible 확인
  ansible --version

 2) ssh-key 생성
 # ssh-key 생성
  ssh-keygen

 # key 원격서버에 복사
  ssh-copy-id  [원격서버계정]@[원격서버 IP]
    ssh-coppy-id root@127.0.0.1

 3) 인벤토리파일 작성
  /etc/ansible/hosts
   vi /etc/ansible/hosts

    인벤토리 목록 서버 접속확인
     ansible all -m ping


명령어
 # ping
  ansible [서버그룹] -m ping


## 참조 URL
https://5equal0.tistory.com/entry/Ansible-%EC%95%A4%EC%84%9C%EB%B8%94Ansible-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%84%A4%EC%B9%98%EC%82%AC%EC%9A%A9%EB%B2%95-w-CentOS-76

https://ichi.pro/ko/ansible-playbook-eul-sayonghayeo-yeogbanghyang-peulogsi-guseong-mich-guseong-pail-jadong-eobdeiteu-206504986767029
