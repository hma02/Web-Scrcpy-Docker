# Web-Scrcpy Docker

> [!IMPORTANT]
> 请注意，Werkzeug 不支持多线程并发，请使用单线程逻辑使用 Web 界面。  
> 不建议反代到公网，建议使用 VPN 或虚拟局域网等方式进行访问。

相对于上游，添加了链接时自动镜像与本地保存设备列表功能，移除了 AI 聊天与自动操作功能。  
使用 Docker 镜像在 `x86_64` 架构的 Linux 系统上运行；或使用源码形式在 `x86_64` 架构的系统上运行。

## 快速开始

### 使用 Docker Run 命令

1. 拉取镜像

   ```bash
   bash build.sh
   ```

### 使用 Docker Compose

   ```bash
   web-scrcpy:
      container_name: web-scrcpy
      image: web-scrcpy:test
      restart: unless-stopped
      ports:
         - "5001:5000"
      stdin_open: true
      tty: true
      volumes:
         - /home/pi/.android:/root/.android   # use the same key as host, assume host already connected those devices
      command: >
         sh -c "
         . /app/venv/bin/activate &&
         adb connect 192.168.1.XX1:5555 &&
         adb connect 192.168.1.XX2:5555 &&
         adb connect 192.168.1.XX3:5555 &&
         python3 app.py --port 5000 --video_bit_rate 256000
         "
   ```


```bash

docker-compose up -d
```

### 在本地构建 Docker 镜像

1. 克隆仓库

   ```bash
   cd web-scrcpy
   ```

### 源码运行

1. 安装依赖

   ```bash
   pip install -r requirements.txt
   ```

2. 运行服务

   ```bash
   python app.py
   # 或指定视频码率
   python app.py --video_bit_rate 1024000
   ```

3. 访问 Web 界面
   - 打开浏览器访问 `http://localhost:5000/`。

### 配置文件

- `data/.env`：环境变量配置文件。
  - `ADB_DEVICES`：adb 设备列表，格式为 `{"设备名称": "IP:PORT"}`。
  - `AUTO_STOP_TIME`：自动停止镜像时间，单位为分钟。默认值为 15 分钟。
  - `DEMO_MODE`：演示模式开关，值为 `true` 或 `false`。默认值为 `false`。

### 演示模式

功能：

- 禁止用户在"链接新设备"中输入
- 只允许通过已保存的设备进行链接
- 不显示"已保存的设备"的"删除选项"

## 许可证

Apache License 2.0
