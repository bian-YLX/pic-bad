# 使用 ssh 服务在 vs 远程连接 wsl 的方法

## 注：想要长期不更改 VS 配置文件的 IP 需要配置静态 IP（ip变动时会断开连接，需要更改 config 文件 IP）。

### 1.下载 vs 扩展包 

#### 在 vs 扩展商店搜索 Remote Development 并下载

![image-20250926101008011](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926101008011.png)

### 2.**在 WSL 中安装并启动 SSH 服务**

#### 2.1查看 ssh 是否安装

##### 打开 PowerShell 或 cmd 命令行输入：

```bash
# 查看 ssh 服务是否安装
ssh -V
```

##### 已安装

![image-20250926102443041](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926102443041.png)

#### 2.2安装 ssh

```bash
# 更新包管理器
sudo apt update && sudo apt upgrade -y

# 安装 OpenSSH 服务端
sudo apt install openssh-server -y

# 启动 SSH 服务
sudo service ssh start

# 设置开机自启（可选，避免每次重启 WSL 都要手动启动）
sudo systemctl enable ssh

#查看 ssh 服务状态
sudo service ssh status
```

![image-20250926102801645](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926102801645.png)

### 3.配置 WSL 的 SSH 服务

#### **修改 SSH 配置（解决端口和权限问题）**

编辑 SSH 配置文件：

```bash
sudo vim /etc/ssh/sshd_config
```

确保以下配置正确（若不存在则添加，若被#号注释删去即可）：

```bash
# 端口号（默认22，若被占用可改为其他端口如2222）
Port 22

# 允许密码认证（首次连接可用，后续可改为公钥认证）
PasswordAuthentication yes

# 监听所有地址
ListenAddress 0.0.0.0
```

![image-20250926103319392](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926103319392.png)

![image-20250926103713251](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926103713251.png)

### 4.配置 vs 的 ssh 配置文件并连接

#### 4.1在 wsl 中查看当前ip地址

```bash
#查询当前 wsl 网络 ip
ip addr
```

![image-20250926105200023](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926105200023.png)

##### 若无需外部通信可连接本地回环地址：

![image-20250926135958667](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926135958667.png)

#### 4.2修改 vs 中 ssh 的配置文件



```bash
Host BMY					#定义一个名为 "BMY" 的连接配置
  HostName 192.168.212.25 	# wsl 网络 ip 地址
  User bian					#登录 wsl 使用的用户名		
  Port 22					# ssh 服务端口（默认就是 22，可省略）
```

![image-20250926105327189](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926105327189.png)

##### 配置完成后保存。

#### 4.3通过vs的远程资源管理器连接

![image-20250926110426593](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926110426593.png)

##### 输入连接命令。

```bash
#ssh连接命令
ssh bian@BMY #ssh （用户名）@（主机名）
```

![image-20250926110531463](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926110531463.png)

##### 输入用户密码后即可连接成功

### 5.优化：配置公钥认证（免密码登录）

#### **5.1在 WSL 中生成密钥对（若已存在可跳过）**

```bash
ssh-keygen -t ed25519  # 一路回车默认生成即可
```

#### **5.2将公钥添加到授权列表**

```bash
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys  # 确保权限正确
chmod 700 ~/.ssh                  # 确保.ssh目录权限正确
```

#### **5.3将 WSL 私钥复制到 Windows（可选）**

##### 若 VS 需要使用 Windows 环境的密钥，可将 WSL 私钥复制到 Windows 的 `.ssh` 目录：

```bash
cp ~/.ssh/id_ed25519 /mnt/c/Users/你的Windows用户名/.ssh/
```

##### 然后在 VS 的 SSH 配置文件中指定密钥路径：

![image-20250926104047786](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926104047786.png)

#### 5.4配置 wsl 中 ssh 服务器查找公钥路径（远程授权列表）

编辑 SSH 配置文件：

```bash
sudo vim /etc/ssh/sshd_config

#配置指定了 SSH 服务器在验证密钥时会查找的公钥文件路径
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
```

![image-20250926105105679](C:\Users\MACHENIKE\AppData\Roaming\Typora\typora-user-images\image-20250926105105679.png)

##### 配置完成后直接连接即可，此时无需输入用户密码。