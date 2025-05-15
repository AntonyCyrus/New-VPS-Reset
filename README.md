1.更新系统配置
```bash
sudo apt update && sudo apt upgrade -y && sudo apt full-upgrade -y && sudo apt autoremove -y && sudo apt clean
```

2.一键dd操作系统（此处使用 @bin456789 提供的脚本）
下载一键dd脚本
```bash
curl -O https://raw.githubusercontent.com/bin456789/reinstall/main/reinstall.sh || wget -O reinstall.sh $_
```
唤起指令介绍
```bash
bash reinstall.sh
```
dd系统指令为
```bash
./reinstall.sh <系统名> <版本号> [选项]
```
以Debian 12系统举例
重装系统为Debian 12同时设置SSH密码为MyNewPass123
```bash
bash reinstall.sh debian 12 --password MyNewPass123
```
以Windows为例
重装系统为Windows_Server_2022英文版本同时设置SSH密码为MyWinPass2022以及RDP端口为3389
```bash
bash reinstall.sh windows --image-name "windows 2022" --iso "https://software-download.microsoft.com/download/pr/Windows_Server_2022_English.iso" --password MyWinPass2022 --allow-ping --rdp-port 3389
```

3.使用新的SSH密码进入系统然后运行一次更新
```bash
sudo apt update && sudo apt upgrade -y && sudo apt full-upgrade -y && sudo apt autoremove -y && sudo apt clean
```

4.在本地生成SSH端对端加密密钥对
推荐生成ED25519类型
生成Passphrase
生成Cipher,首选chacha20-poly1305@openssh.com，其次aes-256-ctr或aes-128-ctr

5.创建并上传公钥
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```
粘贴你的公钥进去
修改公钥文件修改权限
```bash
chmod 600 ~/.ssh/authorized_keys
```

6.禁用密码登陆
编辑配置文件
```bash
sudo nano /etc/ssh/sshd_config
```
修改一下内容
```bash
PermitRootLogin prohibit-password
PasswordAuthentication no
PubkeyAuthentication yes
```
重启SSH服务
```bash
sudo systemctl restart sshd
```
如果遇到报错可用指令排查配置文件语法
```bash
sshd -t
```
检查是否启用了密码登录
```bash
grep -i passwordauthentication /etc/ssh/sshd_config
```
输出
```bash
PasswordAuthentication no
```
即表示配置正确

7.给密钥文件上锁
```bash
sudo chattr +i ~/.ssh/authorized_keys
```
如果需要修改公钥文件则需要解锁
```bash
sudo chattr -i ~/.ssh/authorized_keys
```
验证密钥文件不可修改
```bash
echo "# test" >> ~/.ssh/authorized_keys
```
输出类似
```bash
-bash: /root/.ssh/authorized_keys: Operation not permitted
```
即表示设置成功
