# Domain Seeker (海量扫描优化版)

一个高效、灵活的域名可注册性检测工具，基于 RDAP 协议检查多种顶级域名 (TLD) 的可用性。

此 Fork 版本特别针对**海量批量扫描（如穷举 3-4 位纯字母组合）**进行了配置优化，完美适配 Debian/Ubuntu VPS 运行环境，并内置了便捷的一键启停后台管理脚本。

## ✨ 核心优化与功能特点

- **突破海量扫描瓶颈**：内置自定义 Python 生成器（`generator_func.py`），利用生成器 (`yield`) 极低内存占用，轻松应对数十万乃至百万级域名穷举。
- **防封禁策略**：优化了针对 `.de` 等严格注册局的查询延迟保护。
- **进程守护与一键管理**：内置 `start.sh` 和 `stop.sh`，全后台静默运行，断开 SSH 终端绝不中断任务。
- **无缝部署环境**：提供绕过最新 Linux 系统 `pip` 保护机制的强制安装方案。

---

## 🚀 快速部署 (Debian / Ubuntu VPS 环境)

### 1. 克隆项目
```bash
git clone [https://github.com/oldxianyu/domain-scanner-4-nodeseeker.git](https://github.com/oldxianyu/domain-scanner-4-nodeseeker.git)
cd domain-scanner-4-nodeseeker

```

### 2. 配置 Python 环境与安装依赖

在较新的 Linux 系统（如 Debian 12+ / Ubuntu 23.04+）中，系统会保护全局 Python 环境。推荐使用以下命令强制安装依赖（适合 VPS root 用户）：

```bash
apt update
apt install python3-pip python3-venv -y

# 强制安装项目所需的依赖包（绕过系统环境保护）
pip3 install -r requirements.txt --ignore-installed --break-system-packages

```

# 生成启动脚本 start.sh
```
cat << 'INEOF' > start.sh
#!/bin/bash
PID=$(pgrep -f "python3 main.py")
if [ -n "$PID" ]; then
    echo "⚠️ 提示: 扫描程序已经在运行中 (PID: $PID)，请勿重复启动！"
else
    echo "🚀 正在启动 Domain Seeker 后台扫描..."
    python3 main.py
    echo "✅ 启动成功！输入 tail -f log.txt 查看进度。"
fi
INEOF
```
# 生成停止脚本 stop.sh
```
cat << 'INEOF' > stop.sh
#!/bin/bash
PID=$(pgrep -f "python3 main.py")
if [ -z "$PID" ]; then
    echo "❌ 提示: 未找到运行中的扫描程序。"
else
    echo "⚙️ 正在停止扫描进程 (PID: $PID)..."
    kill -9 $PID
    echo "✅ 成功: 扫描程序已彻底停止！"
fi
INEOF
```

### 3. 赋予脚本执行权限

```bash
chmod +x start.sh stop.sh

```
---

## ⚙️ 使用方法与核心配置

本项目支持两种扫描模式，请根据需求修改 `config.txt` 和数据源。

### 场景 A：使用字典文件（少量扫描）

如果你有几百、几千个特定单词想扫描：

1. 编辑 `domains.txt`，每行填入一个域名基础部分（不含后缀，例如 `example`）。
2. 打开 `config.txt`，将 `domain_source` 设为 `file`。

### 场景 B：使用生成器函数（默认推荐，海量穷举扫描）

本项目默认已配置为穷举所有 3-4 位的纯字母组合（近 50 万个域名）。

**1. 域名生成规则 (`generator_func.py`)**
如需修改组合逻辑（例如加入数字或更改长度），请编辑此文件：

```python
import itertools

def generate_domains():
    chars = 'abcdefghijklmnopqrstuvwxyz'
    
    # 生成 3 位字母组合
    for item in itertools.product(chars, repeat=3):
        yield ''.join(item)
        
    # 生成 4 位字母组合
    for item in itertools.product(chars, repeat=4):
        yield ''.join(item)

```

**2. 核心扫描配置 (`config.txt`)**
进行海量扫描时，**务必保持查询延迟配置**，否则极易触发注册局 429 报错从而封禁服务器 IP：

```ini
tlds = .de
domain_source = generator
delay = 2.0  # 海量扫描强烈建议设置 2.0 秒以上
max_retries = 3
enable_upload = false # 关闭自动上传失效接口，结果自动保存在本地

```

---

## 🛠️ 一键运行与监控后台

配置完成后，你可以使用内置的快捷脚本轻松管理任务：

* **🚀 启动后台扫描**：
```bash
./start.sh

```


* **📊 查看实时进度**：
```bash
tail -f log.txt

```


*(按 `Ctrl + C` 退出查看日志，不会影响后台运行)*
* **🎉 查看成功结果**（空闲可注册的域名）：
```bash
cat results.txt

```


* **🛑 彻底停止扫描**：
```bash
./stop.sh

```



---

## 📋 状态码说明

在 `log.txt` 中，工具使用 HTTP 状态码判断域名可用性:

* **404**: 域名未注册，通常可注册（已自动记录入 `results.txt`）。
* **200/401**: 域名已注册或不可注册。
* **429**: 查询过于频繁，触发速率限制（**遇到此代码，请立刻使用 `./stop.sh` 停止程序，并调大 `config.txt` 中的 `delay` 参数**）。

---

## 📄 许可证

MIT License

