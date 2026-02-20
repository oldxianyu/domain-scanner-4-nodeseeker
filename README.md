```markdown
# Domain Seeker (海量扫描优化版)

一个高效、灵活的域名可注册性检测工具，基于 RDAP 协议检查多种顶级域名 (TLD) 的可用性。此 Fork 版本特别针对**海量批量扫描（如穷举 3-4 位纯字母组合）**进行了配置优化与 VPS 运行环境适配。

## 功能特点

- 使用现代 RDAP 协议进行域名查询，比传统 WHOIS 更快速可靠。
- 支持多种顶级域名 (TLD) 检测（如 `.de`, `.com` 等）。
- 两种域名获取方式：文件导入 或 **自定义 Python 生成函数（极低内存占用，适合百万级扫描）**。
- **全后台静默运行**，自带守护进程 (Daemon)，断开 SSH 终端不中断任务。
- 扫描结果直接保存在本地，或通过浏览器打开 URL 查看。

---

## 安装方法 (Debian / Ubuntu VPS 环境)

### 1. 克隆项目到本地
```bash
git clone [https://github.com/oldxianyu/domain-scanner-4-nodeseeker.git](https://github.com/oldxianyu/domain-scanner-4-nodeseeker.git)
cd domain-scanner-4-nodeseeker

```

### 2. 配置 Python 环境与安装依赖

在较新的 Linux 系统（如 Debian 12+ 或 Ubuntu 23.04+）中，系统会保护全局 Python 环境。推荐使用以下命令强制安装依赖（适合 VPS root 用户）：

```bash
# 更新并安装 pip
apt update
apt install python3-pip python3-venv -y

# 强制安装项目所需的依赖包（绕过系统环境保护）
pip3 install -r requirements.txt --ignore-installed --break-system-packages

```

---

## 使用方法与核心配置

使用前，你需要配置 `config.txt` 和数据源（`domains.txt` 或 `generator_func.py`）。

### 场景 A：使用字典文件（少量扫描）

如果你有几百、几千个特定单词想扫描：

1. 编辑 `domains.txt`，每行填入一个域名基础部分（不含后缀，例如 `example`）。
2. 打开 `config.txt`，将 `domain_source` 设为 `file` 或 `auto`。

### 场景 B：使用生成器函数（海量穷举扫描）

如果你想扫出所有 3 到 4 位的纯字母组合（近 50 万个域名），不能使用文本文件，需要使用生成器脚本。

**第一步：配置生成器 `generator_func.py**`
清空原有内容，填入以下代码（利用 `itertools` 避免内存溢出）：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import itertools

def generate_domains():
    """生成所有 3 到 4 位纯字母组合的域名"""
    chars = 'abcdefghijklmnopqrstuvwxyz'
    
    # 生成 3 位字母组合 (17,576 个)
    for item in itertools.product(chars, repeat=3):
        yield ''.join(item)
        
    # 生成 4 位字母组合 (456,976 个)
    for item in itertools.product(chars, repeat=4):
        yield ''.join(item)

```

**第二步：修改 `config.txt` (防封禁配置)**
进行海量扫描时，**务必调高查询延迟**，否则极易触发注册局 429 报错封禁 IP：

```ini
# 要扫描的顶级域名列表，例如 .de
tlds = .de

# 强制指定为 generator，去读取我们的 Python 脚本
domain_source = generator

# 查询间隔(秒)。海量扫描强烈建议设置 2.0 秒以上
delay = 2.0

# 查询失败时的最大重试次数
max_retries = 3

# 关闭自动上传结果（避免原图床/展示服务失效导致报错，结果会保存在本地）
enable_upload = false

```

---

## 运行与监控

### 1. 启动扫描

```bash
python3 main.py

```

*提示：启动后程序会自动转入后台运行，你可以安全地关闭终端，程序将持续工作直到扫描完成。*

### 2. 实时查看进度

想看它正在测试哪个域名，或者是否被限制频率（429 错误），可以实时查看日志：

```bash
tail -f log.txt

```

*(按 `Ctrl + C` 退出查看，不会打断后台扫描)*

### 3. 查看成功结果

空闲可注册的域名会自动追加保存到结果文件中：

```bash
cat results.txt

```

### 4. 强制停止扫描

如果需要中途停止任务：

```bash
# 查找主程序的 PID
ps aux | grep main.py

# 强制杀掉进程（将 PID 替换为你看到的数字）
kill -9 <PID>

```

---

## 状态码说明

在 `log.txt` 中，工具使用 HTTP 状态码判断域名可用性:

* **404**: 域名未注册，通常可注册（记录入 `results.txt`）。
* **200/401**: 域名已注册或不可注册。
* **400**: 无效请求，包含非法字符或格式错误。
* **429**: 查询过于频繁，触发速率限制（**遇到此代码请停止程序并调大 `delay` 参数**）。

---

## 使用 AI 辅助生成自定义函数

你可以使用 AI 工具帮助编写更复杂的规则。只需发送如下提示词：

> 请编写一个名为 `generate_domains` 的 Python 函数，使用生成器 (`yield`) 返回符合以下条件的域名基础部分(不含 TLD)：
> * [例如：长度为5个字符，包含至少2个数字，数字不能相邻]
> 
> 
> 要求：不接受参数，必须使用 `yield` 返回每个域名，不含后缀，代码高效不产生重复。只返回代码。

---

## 许可证

MIT License
