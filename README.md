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

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/bin:/usr/games

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Include /etc/ssh/sshd_config.d/*.conf

Port 12783
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

LoginGraceTime 30s
PermitRootLogin prohibit-password
#StrictModes yes
MaxAuthTries 2
#MaxSessions 10

PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to "no" here!
PasswordAuthentication no
#PermitEmptyPasswords no

# Change to "yes" to enable keyboard-interactive authentication.  Depending on
# the system's configuration, this may involve passwords, challenge-response,
# one-time passwords or some combination of these and other methods.
# Beware issues with some PAM modules and threads.
KbdInteractiveAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin prohibit-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

# no default banner path
#Banner none

# Allow client to pass locale and color environment variables
AcceptEnv LANG LC_* COLORTERM NO_COLOR

# override default of no subsystems
Subsystem	sftp	/usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#	X11Forwarding no
#	AllowTcpForwarding no
#	PermitTTY no
#	ForceCommand cvs server

ChallengeResponseAuthentication no

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
