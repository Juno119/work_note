# ubuntu 系统遇到的问题及解决方法   

## 目录  
- [1. sublime-text 环境配置相关](./doc/sublime-text3)   
- [2. git 使用 skills ](./doc/git_usage/)  
 
## 常用命令   
- [ubuntu 常用命令 cheatsheet](./doc/quick_cmd.md)  
- [vi 使用教程](./doc/vi_usage.md)  
- [grep 用法](./doc/grep_usage.md)   
- [nohup 的使用](./doc/nohup.md)   
 
## 安装系统必备    
- [更换 ubuntu 软件下载源(163)](./doc/sourceslist.md)  
- [修改系统默认启动 ubuntu 或 windows](./doc/default_grub.md)  
- [pip install 更换源](./doc/pip_install_source.md)  
- [搜狗输入法的安装和卸载](./doc/sogou_install.md)  
- [.bashrc 文件的配置](./doc/bashrc_config.md)  
- [鼠标右键添加'在当前位置打开终端'](./doc/open_termials.md)  
- [配置 shadowsocks](./doc/shadowsocks_install.md)  
- [ssh-keygen 生成步骤](./doc/ssh_keygen.md)  
- [jupyter-python2.7 安装问题](./doc/jupyter_python2.7_install.md) 
- [Ubuntu 16.04 系统备份和恢复](./doc/system_backup_recover.md)  
- [在 virtual box 中安装 windows10](./doc/install_windows_in_virtualbox.md)  
- [在 ubuntu 中同时安装 opencv 3 和 opencv 2](./doc/install_opencv2_and_opencv3.md)  
- [在 ubuntu 中安装 meshLab](./doc/meshlab.md)  
 
## 系统使用中的坑   
- [chrome 提示无法正确打开您的个人资料](./doc/chrome.md)  
- [ubuntu 14.04 状态栏不显示时间](./doc/timedate_bar.md)  
- [ubuntu 14.04 设置程序启动快捷键](./doc/shortcuts.md)  
- [更改 ubuntu 启动等待时间](./doc/grub_timeout.md)  
- [ubuntu 安装 Java1.7](./doc/java1.7_install.md)  
- [如何让普通用户访问 /dev/ttyUSB0 ](./doc/minicom_permision.md)  
- [搭建 iTop4412 使用的 NFS 环境 ](./doc/nfs.md)  
- [NVIDIA GeForce 720M 驱动安装](./doc/nouveau_nvidia.md)  
- [修改命令提示符 PS1 ](./doc/ps1_modify.md)  
- [ubuntu ss5 polipo 全局代理](./doc/ss5-polipo_proxy.md)  
- [system settings 里面选项丢失](./doc/system_setting.md)  
- [ubuntu 终端闪退](./doc/terminals_crash.md)  
- [python-apt 出错](./doc/no_module_named_apt_pkg.md)  
- [改变系统中的默认的 python 版本](./doc/change_python_version_in_system.md)   
- [ubuntu 查看内核版本](./doc/ubuntu_kernel_version.md)  
- [chrome 推送网页内容到 kindle ](./doc/send_chrome_to_kindle.md)  
- [安装和使用 Docker ](./doc/docker_install.md)  
- [查看目录占用的磁盘空间](./doc/disk_space_usage.md)   
- [qt 无法使用搜狗输入法](./doc/qt_sogou.md)   

```shadowsocks5
{
	"server": "104.128.95.11",
	"server_port": 443,
	"local_address" : "127.0.0.1",
	"localPort": 1080,
	"password": "MTc5NjJkNj",
	"timeout": 600,
	"method": "aes-256-cfb",
	"fast_open": false
}
```

```
{
	"server": "23.83.244.29",
	"server_port": 444,
	"local_address" : "127.0.0.1",
	"localPort": 1080,
	"password": "NmEyOTg5Nm",
	"timeout": 600,
	"method": "aes-256-cfb",
	"fast_open": false
}
```

好像已经不可用   
```
{
	"server": "69.171.73.29",
	"server_port": 444,
	"local_address" : "127.0.0.1",
	"localPort": 1080,
	"password": "tgKqnWART4",
	"timeout": 600,
	"method": "aes-256-cfb",
	"fast_open": false
}
```