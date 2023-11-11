SSH 是一种加密的网络传输协议，主要用途是远程登录系统，并提供一个交互式的虚拟终端或者直接远程执行命令。SSH 还能扮演类似 SSL 的角色，通过在网络中创建安全隧道，实现对客户端与服务端之间的数据加密。

[OpenSSH](https://www.openssh.com/) 是 SSH 的开源实现，Windows 10 1803及后续版本都默认启用了 OpenSSH，相关命令行工具存储于`C:\Windows\System32\OpenSSH`目录。工具可分为三类，简介如下：

1. 远程操作类

   - ssh—OpenSSH 终端连接工具
   - scp—文件远程拷贝工具
   - sftp—OpenSSH 版 FTP

2. 密钥管理类

   - ssh-add—将私钥添加到 ssh-agent 的代理缓存
   - ssh-keygen—生成、管理和转换密钥
   - ssh-keyscan—获取远程主机的 OpenSSH 公钥

3. 服务类

   - sshd—OpenSSH 服务守护进程

   - ssh-agent—OpenSSH 私钥管理工具

**以下部分操作只适用于 Windows 系统登录 Linux 系统 ssh 的场景！！！**

## 创建密钥

ssh 密钥登录无需输入密码，更加方便且杜绝了暴力破解的风险。命令行中使用 ssh-keygen 工具生成公钥/私钥对，一路回车使用缺省参数即可，Windows 中生成的密钥默认保存到`C:\Users\[用户名]\.ssh`目录。生成的私钥文件是`id_rsa`，公钥文件是`id_rsa.pub`，pub即 public，公开的、公用的意思。

## 部署公钥

在 .ssh 目录下打开命令行，然后执行下列组合命令将刚生成的公钥文件上传到服务器登录用户的家目录。完成上述操作后，重新登录 ssh 就无需输入密码了。

```bash
C:\Users\apple\.ssh> type id_rsa.pub | ssh dog@192.168.0.5 "mkdir -p .ssh && cat >> .ssh/authorized_keys && chmod 600 .ssh/authorized_keys"
```

## 其它设置

### 免用户名登录

如果想进一步发扬程序员的三大美德，登录时连用户名都省掉，岂不更美哉？

首先在服务端新建一个与客户端同名的账户`useradd -m -s /bin/bash apple && passwd apple`，然后再将其它账户的`.ssh`文件夹拷贝到新账户的家目录`cp -r /home/dog/.ssh /home/apple`。出于安全考虑，建议重新生成秘钥，这里只是演示下过程。上述操作用 root 账户完成，拷贝过来的文件夹的所有者自然也是 root，还需将所有者变更为新账户`chown -hR apple /home/apple/.ssh`。好了，现在可以直接用主机地址登录了`ssh 192.168.0.5`。

### putty自动登录

虽然中文版 putty 出现过严重的后门事件，且 Windows Terminal 用起来也很舒服，但是putty仍有相当部分的拥趸。使用以上生成并部署完的密钥对，只需两步即可实现 putty 免密码登录。

1. 转换公钥

   OpenSSH 与 putty 的密钥格式不同，使用 puttygen工具将 ssh-keygen 生成的私钥转成 putty 格式。点击`Load`按钮，选择私钥文件`id_rsa`，然后再保存成新的私钥文件。

   ![image-20211009222032353](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20211009222032353.png)

2. 加载公钥

   putty 设置窗口的 Auth 标签页中选择转换好的 ppk 私钥文件，再填写默认登录账户、主机地址等信息，最后将当前设置保存成一个会话，下次双击会话名即可自动登录。

   ![image-20211009222400149](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20211009222400149.png)

