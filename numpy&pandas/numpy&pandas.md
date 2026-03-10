### anaconda+jupyter：新的开发环境

#### anoconda、jupyter环境

Anaconda是一个**专注于数据科学和机器学习领域的 Python（和R语言）发行版**，它的核心目标是提供一个“开箱即用”的、稳定且高效的开发环境。它就像一个全能工具箱，让你不必再为安装和管理各种复杂的软件包而烦恼。Anacconda可以直观地理解为：**Anaconda = 解释器 + 依赖管理工具 (Conda) + 预安装工具包 (Python/R + 常用库) + 集成工作台 (Navigator/Jupyter**等)。  

`Jupyter`是一个庞大的开源项目，其名字源自它最初支持的三种核心编程语言：Julia、Python 和 R。如今，它已发展为一个支持超过100种编程语言的交互式计算平台 。它的核心是**计算叙事**的理念，即将`代码`、运行结果、`可视化图表`和`解释性文本`融合在一个单一文档——**Notebook** 中

- `Jupyter Notebook` (经典界面)：这是最初的Web应用程序，提供了简洁、以文档为中心的体验，非常适合创建和分享包含代码、方程、可视化和叙述性文本的计算文档 。
- `JupyterLab` (下一代界面**)**：作为当前更`主流的开发环境`，JupyterLab 提供了一个功能更强大、布局更灵活的集成开发环境。你可以同时打开 Notebook、代码文件、终端、数据文件等，并随意拖放排列，就像 IDE 一样 。
- `JupyterHub`：这是一个`多用户版本`，允许组织为成百上千的用户（如公司团队、教室里的学生）集中部署和管理 Jupyter 环境，每个人都能通过浏览器访问自己的专属服务器 。
- Jupyter 内核 (`Kernels`)：内核是 Notebook 的心脏，它是一个独立的进程，负责执行用户编写的代码，并将结果返回给界面。Jupyter 的强大之处在于其语言无关性，你可以通过安装不同内核（如 `IRkernel` 用于 R 语言），在同一个 Notebook 界面中编写和运行多种语言

Jupyter 之所以能在算法和数据开发中占据核心地位，源于它完美契合了这一领域**探索性、迭代性和协作性**的工作特点。

安装步骤

[anaconda | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)

<img src="assets/image-20260311051137510.png" alt="image-20260311051137510" style="zoom: 50%;" /> 

#### conda常用命令

<img src="assets/image-20260311043331608.png" alt="image-20260311043331608" style="zoom:67%;" />  

![image-20260311043355288](assets/image-20260311043355288.png)

![image-20260311043434984](assets/image-20260311043434984.png)  

#### Jupeter常用快捷键和基本使用

- Esc进入命令模式，Enter进入当前cell的编辑模式

- 点击“键盘”图标可以打开命令面板，可以利用该面板执行操作

  ![image-20260311033941763](assets/image-20260311033941763.png)

- 命令模式

  - Ctrl + Enter运行cell，Shift + Enter运行cell并选中下面一个cell，Alt + cell运行cell并在下方插入一个新cell
  - a在上方插入cell，b在下方插入cell，dd删除当前cell
  - m将cell设置为markdown模式，y将cell设置为代码模式
  - space/shift+space 向下/向上滚动页面
  - h查看快捷键帮助

- 编辑模式

  - tab 缩进或代码补全
  - Ctrl + ] 缩进选中的代码，Ctrl + [ 向左缩进选中的代码
  - ctrl + shift + -，在当前光标位置将cell分为两个cell
  - shift + tab，把光标放在函数调用处的后面或参数位置，可以查看该函数的文档

#### 配置远程jupyter服务、继承pycharm

一般把jupyter server放在linux环境上，就jupyerhub而言，linux环境下的不同用户远程登录jupyter会分配不同的运行空间，对应**不同的`kernal`进程**（py进程），这样做有很多好处

- 权限控制明晰，一般而言用户就只能在家目录工作
- linux环境本身支持和部署环境一致
- 支持数据直接操作和大数据组件生态
- 资源控制清晰，给每个进程分配配额的资源
- 不同kernal网络端口隔离

在linux安装好anaconda后用`jupyter lab`命令启动服务后，默认只运行本地访问，因此运行前有必要做一下准备

```bash
# 生成jpt配置文件，~/.jupyter/jupyter_notebook_config.py
jupyter server --generate-config

# 生成密码
conda activate your_env_name # 激活环境
from jupyter_server.auth import passwd
passwd() # 生成密码得到一串哈希值，

# 在配置文件添加以下内容，根据自己是jupyter notebook还是lab自行修改前缀
# 允许远程访问
c.ServerApp.allow_remote_access = True
# 监听所有网络接口，允许任何IP连接
c.ServerApp.ip = '0.0.0.0'
# 启动时不要自动打开浏览器
c.ServerApp.open_browser = False
# 设置你刚刚生成的密码哈希值
c.PasswordIdentityProvider.hashed_password = '粘贴你生成的argon2密码哈希值'
# （可选）设置JupyterLab的启动目录，例如 /home/your_username
c.ServerApp.root_dir = '/home/your_username'
# （可选）指定一个端口，例如 8888
c.ServerApp.port = 8888
```

如果是jpt hub环境，有必要设置开机自启动，新建文件/etc/systemd/system/jupyter.service

```bash
[Unit]
Description=Jupyter lab (Anaconda)
After=network.target

[Service]
Type=simple
User=linuxuser
WorkingDirectory=/home/linuxuser
# 关键：通过bash激活conda环境后再启动jupyter
ExecStart=/bin/bash -c ". anaconda3安装位置/etc/profile.d/conda.sh && conda activate 环境名称 && exec jupyter lab --config=/home/用户名称/.jupyter/jupyter_notebook_config.py"
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
systemctl enable jupyter
```

必要地，还需为jupyter服务提供SSL保护，并将不同的conda env注册到jupyter kernal。具体做法不做详细展开。

```bash
# 修改jupyter_notebook_config.py
# 指定私钥文件路径
c.JupyterHub.ssl_key = '/path/to/your/private.key'
# 指定证书文件路径
c.JupyterHub.ssl_cert = '/path/to/your/certificate.crt'
```

```py
# 激活环境后设置到jpt kernal
python -m ipykernel install --user --name="env_name" --display-name="kernal_name"
```

![image-20260311045943163](assets/image-20260311045943163.png) 