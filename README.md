# VSCode ROS2 Workspace Template

This template will get you set up using ROS2 with VSCode as your IDE.

A fork of [athackst/vscode_ros2_workspace](https://github.com/athackst/vscode_ros2_workspace) with better wsl2 support.

## Setup Tutorial

### Prerequisite

1. **系统盘下至少有 5G 的空余空间**。否则可能安装失败。  
2. 由于网络原因，可能需要提前配置好系统代理。  

### Setup docker

#### Windows - WSL2

1. 使用管理员权限打开终端

2. 在打开的终端中运行：
	```powershell
	wsl --install Ubuntu-22.04
	```

3. 安装 Docker Desktop。

	```powershell
	winget install Docker.DockerDesktop
	```

4. 打开 Docker Desktop，跳过登录和那些麻烦的选择。

> [!TIP]  
> Docker 服务会随 Docker Desktop 启动。启动完毕后可关闭 Docker Desktop，保留 Docker 服务在后台即可。

#### Linux

See [docker docs](https://docs.docker.com/engine/install/).

### Open the repo in vscode

1. 使用 vscode 打开本项目目录   

   Now that you've cloned your repo onto your computer, you can open it in VSCode (File->Open Folder).   

2. 安装 vscode 插件 `Dev Container`

   按下快捷键 `Ctrl+P` 调出 Vscode Quick Open，输入：
   ```
   ext install ms-vscode-remote.remote-containers
   ```

3. 启动 DevContainer

   按下快捷键 `Ctrl+Shift+P`，输入 `rebuild`，选择 `Dev Containers: Rebuild and Reopen in Container`

> [!NOTE]  
> 请在打开 DevContainer 时关闭代理软件的 TUN 模式。否则会导致 vscode 插件无法自动安装。

4. 等待启动。第一次启动需要构筑镜像，可能会花费长达 10 分钟。

If you open a terminal inside VSCode (Terminal->New Terminal), you should see that your username has been changed to `ros`, and the bottom left green corner should say "Dev Container"

![template_container](https://user-images.githubusercontent.com/6098197/91332895-adbf1500-e781-11ea-8afc-7a22a5340d4a.png)

## Optional Setup

### Configurate USB passthrough (WSL Only)

若你需要在开发容器中使用 USB 设备（如工业相机、串口等），请进行以下操作。
Linux 系统不需要按照此部分操作。可直接使用 USB 设备。

> [!NOTE]  
> 需要更好的解决方案。Docker Desktop 运行于一个基于 Alpine 的 WSL 镜像 `docker-desktop`，而 usbipd-win 不支持直接 attach 到 `docker-desktop` 内。[此 issue](https://github.com/dorssel/usbipd-win/issues/669) 建议安装一个单独的 Ubuntu 22.04 镜像用于接入 USB。能用但不优雅。

1. 下载 usbipd-win >= 4.0
   
   ```
	winget install usbipd
   ```

> [!IMPORTANT]   
> 请确保你的 `usbipd-win` 版本 >= 4.0. 你可以手动从 [GitHub Release](https://github.com/dorssel/usbipd-win/releases) 安装最新版。


2. 安装 Ubuntu-22.04 WSL
   
   ```powershell
   wsl --install Ubuntu-22.04
   ```
   首次运行请等待安装完成。安装完成后会自动切入 WSL Ubuntu-22.04 系统内终端。  
   **保持此窗口在后台运行，直到完成此部分前不要关闭。**

3. 列出设备的 BUSID  
   
   启动一个有管理员权限的新终端，运行：
   ```powershell
   usbipd list
   ```
   输出应当类似于：
   ```
   Connected:
	BUSID  VID:PID    DEVICE                     STATE
	1-3    27c6:589a  Goodix fingerprint         Not shared
	2-3    8087:0032  英特尔(R) 无线 Bluetooth(R)  Not shared
	3-1    2b7e:b557  XiaoMi USB 2.0 Webcam      Not shared
   
   Persisted:
   	GUID                                  DEVICE
   ```
   记住你期望在 Docker 中使用的设备的 BUSID。

4. 设置默认 WSL Distro
   ```powershell
   wsl --set-default Ubuntu-22.04
   ```

5. 配置待穿透设备
   ```powershell
   usbipd bind --busid=<BUSID>
   usbipd attach --wsl --busid=<BUSID>
   ```
   **将你需要在 Docker 环境中使用的设备 BUSID 替换掉命令中的 `<BUSID>`。**  
   例如在上述 `usbipd list` 的输出示例中，你希望将 XiaoMi USB 2.0 Webcam 映射到 Docker 中：
   ```powershell
   usbipd bind --busid=3-1
   ```

> [!TIP]  
> 每次重启宿主机，你需要重新运行第 5 步。


### Setup CUDA

Docker 镜像中已内置 CUDA。不需要进行任何手动配置。  
无需安装 CUDA for WSL2 或在 WSL 中安装 CUDA。  

## Adopt to Your Workflow (暂不对外开放)

1. 在 CodeUp 上 Fork 本项目。

2. 进入 [codeup_circleci_pipeline](https://github.com/cygnomatic/codeup_circleci_pipeline) 配置文件仓库，参考 `.circleci` 目录下 `plain.yml` 新增一个配置文件，修改其中参数。

3. 在 CodeUp 的 设置 -> Webhooks 中增加一个由标签推送事件触发的 Webhook。

4. 进入 CircleCI，在 [CodeUp 项目](https://app.circleci.com/settings/project/circleci/17Q1ozw2K91TaCvtghpJuL/Vx7rcs2Lz2dB8uHjKYDeBb/triggers) 下新增一条 Trigger 指向上一步中新增的 Webhook。

5. 修改 Fork 项目内的 `.devcontainer/requirements/` 中的依赖。

6. 向 CodeUp 的 Fork 推送一个标签，触发 CircleCI 构建镜像并等待构建完成。

7. 修改 `.devcontainer/Dockerfile` 和 `.devcontainer/Dockerfile.l4t35` 中的基础镜像为 CircleCI 构建的镜像。

8. 执行 `Dev Containers: Rebuild and Reopen in Container`。