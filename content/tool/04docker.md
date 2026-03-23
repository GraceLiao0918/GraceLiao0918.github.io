---
title: 🍨 docker
tags:
  - tool
---

# docker安装

## 🧼 下载与安装

1. 访问[Docker官网][https://www.docker.com/products/docker-desktop]
2. 选择`Windows的AMDx64`版本进行下载
3. 双击下载的 Docker Desktop 安装包，启动安装程序
4. 按照屏幕上的指示进行操作，安装完成后，系统会提示重启计算机
5. 启动Docker Desktop
   - 在 Windows 开始菜单中找到 Docker Desktop，点击启动程序
   - 启动过程中，Docker 会自动启动必要的后台服务，并可能会要求你登录 Docker Hub 账户。你可以选择注册一个 Docker Hub 账户，也可以选择跳过此步骤
6. 检查 Docker 运行状态
   - 确保Docker Desktop的图标在系统托盘中显示为正常运行状态（图标为蓝色鲸鱼）

## 🌚 配置国内镜像加速器

- 点击右上角齿轮图标，进入`Settings（设置）`
- 在左侧导航中点击`Docker Engine`
- 修改 `Docker Engine` 配置

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "registry-mirrors": ["https://docker.1ms.run", "https://docker.xuanyuan.me"]
}
```

- 点击右下角的`Apply & Restart（应用并重启）`
- 等待 Docker 重启后配置生效

## 💫 验证 Docker 安装

- 使用命令提示符（cmd）或 Windows PowerShell。

- 输入以下命令，检查 Docker 的版本信息

```powershell
bash
docker --version
```

- 运行一个简单的 Docker 容器，以验证安装是否成功

```powershell
docker run hello-world
```

- 如果安装成功，你会看到 Docker 下载并运行了一个测试容器，显示欢迎消息。

```powershell
Hello from Docker!
This message shows that your installation appears to be working correctly...
```

## 🌬️ 运行STN-Web项目

- 项目地址`https://github.com/camilochs/stnweb`

- 打开docker

- 在项目目录输入`wsl`，打开wsl面板

  ```powershell
  ./docker/create-environment.sh
  ./docker/open-environment.sh
  ```

- 打开给定本地地址进行访问` http://127.0.0.1:8081`
