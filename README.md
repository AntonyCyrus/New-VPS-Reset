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
以Debian 13系统举例
重装系统为Debian 13同时设置SSH密码为MyNewPass123
```bash
bash reinstall.sh debian 13 --password MyNewPass123
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

# ==========================================================
# Optimized sshd_config for Debian VPS
# ==========================================================

# 1. 基础网络设置
Port 12783
AddressFamily inet         # 强制使用 IPv4 (如果不用 IPv6，可提升部分环境下的解析速度)
ListenAddress 0.0.0.0

# 2. 认证加固
LoginGraceTime 30s         # 缩短认证超时
PermitRootLogin prohibit-password
StrictModes yes
MaxAuthTries 3             # 设为 3 比较稳健，防止多个 SSH Key 轮询失败
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
AuthenticationMethods publickey # 显式要求必须公钥认证

# 3. 连接稳定性 (防止连接断开)
TCPKeepAlive yes
ClientAliveInterval 60     # 每 60 秒发送一次心跳
ClientAliveCountMax 3      # 连续 3 次无响应才断开

# 4. 访问限制与转发
X11Forwarding no           # 除非有 GUI 需求，否则设为 no
AllowTcpForwarding yes     # 允许隧道转发（对于内网穿透等场景有用）
PermitTTY yes
MaxSessions 10

# 5. 日志与信息提示
SyslogFacility AUTHPRIV
LogLevel VERBOSE           # 记录更详细的日志（包含指纹），方便排查攻击
PrintMotd no               # 不显示系统默认消息
PrintLastLog yes           # 还是建议保留，方便你监控上次登录是否异常
DebianBanner no            # 隐藏 Debian 特有的版本后缀，减少信息泄露

# 6. 算法性能优化
# 仅允许强算法 (可选，需确保客户端支持，现代 OpenSSH 都支持)
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

# 7. 子系统设置
Subsystem sftp /usr/lib/openssh/sftp-server

# 8. 默认环境
AcceptEnv LANG LC_* COLORTERM NO_COLOR

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
