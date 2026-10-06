# ESPHome 中央空调控制面板

一个可本地部署的 ESPHome 空调网页控制面板。后端通过 ESPHome Native API 建立长连接，自动发现每个设备中的全部 Climate 实体，并实时接收状态推送。一台 ESPHome 可以连接多台空调，也可以同时配置多个 ESPHome 设备。

空调卡片支持开关、模式、温度、风速和别名。每个分组支持全部开启、全部关闭及二次确认。已发现的设备会持久缓存，ESPHome 离线时仍显示卡片并标记为离线。

支持创建自定义分组并将空调卡片直接拖动到目标分组。主页按分组显示独立分区，每个分区可以折叠；没有指定分组的空调统一显示在“未分组”。
每个分组（包括“未分组”）都有独立的“全部开启”和“全部关闭”按钮，操作前会显示仅针对该分组的二次确认。

## ESPHome 端配置

每台设备需要启用 Native API：

```yaml
api:
  encryption:
    key: !secret api_encryption_key
```

面板中填写：

- Native API 主机，例如 `192.168.2.50`，不要带 `http://`
- 端口，默认 `6053`
- `api.encryption.key` 对应的加密密钥；未启用加密时留空

`web_server` 不再是本项目的运行要求，可以从 ESPHome 配置中删除。每台 ESPHome 的主机、端口和 API Key 单独保存；全部设备保存完成后，点击“发现全部子空调”统一扫描所有 ESPHome。发现后由长连接订阅状态；后端再通过 SSE 只向浏览器推送发生变化的空调，浏览器不会每 30 秒拉取并重绘全部卡片。首次连接和 SSE 重连时会完整校准一次状态。配置界面的密钥以明文显示，并以明文保存在本地配置文件中，请仅在可信局域网与可信电脑上使用。

每台 ESPHome 使用独立的稳定标识，因此不同主机下同名的 Climate 不会混淆。删除 ESPHome 设备时，其 Climate 缓存与自定义名称也会删除。

首页使用三个独立 Tab：

- “设备列表”用于查看状态、控制空调和执行分组批量操作；每个分组标题显示空调总数和运行中数量，卡片右上角手柄可用于组内排序或跨组移动
- “设备配置”用于逐台保存 ESPHome 连接并统一发现子空调
- “设备分组”用于创建或删除分组，并通过卡片手柄完成归组和分组内排序；拖动时实时显示落点，顺序会持久保存

页面支持浏览器前进/后退；在配置或分组页按 `Esc` 可返回设备列表。

## Docker 构建与运行

```bash
docker build -t esphome-ac-dashboard:latest .
docker run -d --name esphome-ac-dashboard --restart unless-stopped \
  -p 8080:8080 -v ac-dashboard-data:/data \
  esphome-ac-dashboard:latest
```

或使用 Compose：

```bash
docker compose up -d --build
```

打开 `http://本机IP:8080`。配置保存在 Docker 数据卷 `/data/config.json`。

## Windows EXE

在 Windows 10/11 上安装 Python 3.11 或更新版本，然后执行：

```bat
build-windows.bat
```

生成文件：

```text
dist\ESPHomeACDashboard.exe
```

EXE 包含 Python 运行时、Native API 依赖和网页资源，目标电脑无需安装 Python。双击后会启动本地服务并打开 `http://127.0.0.1:8080`。配置保存在：

```text
%LOCALAPPDATA%\ESPHomeACDashboard\config.json
```

## 本地运行

```bash
python3 -m pip install -r requirements.txt
python3 server.py
```

## 测试数据源

设备配置中的“添加测试数据源（30台）”会添加 `demo://30`，不访问真实硬件，可测试开关、模式、温度、风速以及大规模卡片渲染。后台会让每台测试空调每 10 秒随机刷新一次室温，并以较低概率改变模式、风速和设定温度；同一机组的更新会均匀错开，避免大量卡片在同一时刻刷新。也可以把主机改成 `demo://100` 测试 100 台空调。

## 旧配置迁移

旧版保存的 `http://192.168.x.x` Web Server 地址会自动迁移为 Native API 主机地址，并默认使用端口 `6053`。Web Server 用户名和密码不会用于 Native API；升级后请逐台填写并保存 ESPHome API Key，再统一发现全部子空调。
# esphome-ac-dashboard
# esphome-ac-dashboard
