# Claude Code 配置指南

## 概述
本文档介绍如何配置Claude Code和MCP服务器以支持STM32嵌入式开发。

## 目录
1. [环境要求](#环境要求)
2. [Claude Code安装](#claude-code安装)
3. [MCP服务器配置](#mcp服务器配置)
4. [PlatformIO配置](#platformio配置)
5. [开发环境验证](#开发环境验证)
6. [常见配置问题](#常见配置问题)

## 环境要求

### 硬件要求
- **计算机**: Windows 10/11, macOS, Linux
- **开发板**: STM32F103C8T6 (Blue Pill) 或其他STM32系列
- **调试器**: ST-LINK V2 或兼容调试器
- **USB线**: 用于供电和编程

### 软件要求
- **Claude Code**: 最新版本
- **Rust**: 1.70.0 或更高版本
- **Node.js**: 18.0.0 或更高版本
- **Python**: 3.8 或更高版本
- **Git**: 版本控制

## Claude Code安装

### Windows安装
```bash
# 使用PowerShell安装
winget install Anthropic.ClaudeCode

# 或者从官网下载安装包
# https://claude.com/claude-code
```

### macOS安装
```bash
# 使用Homebrew安装
brew install --cask claude-code

# 或者从官网下载
# https://claude.com/claude-code
```

### Linux安装
```bash
# 下载AppImage
wget https://claude.com/download/claude-code-linux.AppImage
chmod +x claude-code-linux.AppImage
./claude-code-linux.AppImage
```

## MCP服务器配置

### 1. 查看当前MCP配置
```bash
claude mcp list
```

### 2. 配置文件位置
- **Windows**: `C:\Users\<用户名>\.claude\settings.json`
- **macOS**: `~/.claude/settings.json`
- **Linux**: `~/.claude/settings.json`

### 3. 示例配置文件
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-filesystem"]
    },
    "embedded": {
      "command": "npx",
      "args": ["embedded-mcp-server"]
    },
    "platformio": {
      "command": "node",
      "args": ["C:\\Users\\Roland\\.platformio-mcp\\build\\index.js"]
    },
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"]
    }
  }
}
```

### 4. 安装MCP服务器
```bash
# 安装filesystem MCP服务器
npm install -g @modelcontextprotocol/server-filesystem

# 安装embedded MCP服务器
npm install -g embedded-mcp-server

# 安装github MCP服务器
npm install -g @modelcontextprotocol/server-github
```

## PlatformIO配置

### 1. 安装PlatformIO Core
```bash
# 使用pip安装
pip install platformio

# 或者使用独立安装脚本
python -c "$(curl -fsSL https://raw.githubusercontent.com/platformio/platformio/master/scripts/get-platformio.py)"
```

### 2. 验证安装
```bash
pio --version
# 输出: PlatformIO Core, version 6.1.0
```

### 3. 配置PlatformIO MCP服务器
```bash
# 克隆PlatformIO MCP服务器
git clone https://github.com/platformio/platformio-mcp.git
cd platformio-mcp

# 安装依赖
npm install

# 构建项目
npm run build
```

### 4. 测试PlatformIO功能
```bash
# 查看支持的开发板
pio boards stm32

# 创建测试项目
mkdir test_stm32
cd test_stm32
pio project init --board bluepill_f103c8

# 构建项目
pio run
```

## 开发环境验证

### 1. 完整环境检查
```bash
# 检查Claude Code版本
claude --version

# 检查MCP服务器状态
claude mcp list

# 检查PlatformIO
pio --version

# 检查Node.js
node --version

# 检查Python
python --version
```

### 2. 创建测试项目
```bash
# 创建测试目录
mkdir claude-test
cd claude-test

# 创建简单的STM32项目
cat > platformio.ini << EOF
[env:bluepill_f103c8]
platform = ststm32
board = bluepill_f103c8
framework = arduino
EOF

# 创建主程序
mkdir src
cat > src/main.cpp << EOF
#include <Arduino.h>

void setup() {
  pinMode(PC13, OUTPUT);
  Serial.begin(115200);
  Serial.println("Claude Code Test");
}

void loop() {
  digitalWrite(PC13, !digitalRead(PC13));
  delay(1000);
  Serial.println("Toggle LED");
}
EOF
```

### 3. 使用Claude Code构建
```bash
# 通过Claude Code构建项目
# 在Claude Code中执行:
pio run

# 输出应该显示构建成功
# [SUCCESS] Took X.XX seconds
```

### 4. 上传测试
```bash
# 连接STM32开发板
# 通过Claude Code上传
pio run -t upload

# 输出应该显示上传成功
# [SUCCESS] Took X.XX seconds
```

## 常见配置问题

### 问题1: MCP服务器连接失败
**症状**: `claude mcp list` 显示服务器连接失败

**解决方案**:
```bash
# 1. 检查Node.js安装
node --version

# 2. 重新安装MCP服务器
npm uninstall -g @modelcontextprotocol/server-filesystem
npm install -g @modelcontextprotocol/server-filesystem

# 3. 检查配置文件
cat ~/.claude/settings.json
```

### 问题2: PlatformIO命令找不到
**症状**: `pio` 命令无法执行

**解决方案**:
```bash
# 1. 检查Python环境
python --version
pip --version

# 2. 重新安装PlatformIO
pip uninstall platformio
pip install platformio

# 3. 检查PATH环境变量
echo $PATH
```

### 问题3: STM32开发板无法识别
**症状**: 上传时提示找不到设备

**解决方案**:
```bash
# 1. 检查USB连接
# 2. 安装ST-LINK驱动
# 3. 检查设备管理器中的COM端口

# 手动指定端口
pio run -t upload --upload-port=COM3
```

### 问题4: 构建时缺少依赖
**症状**: 构建失败，提示缺少库或工具链

**解决方案**:
```bash
# 1. 清理缓存
pio system prune

# 2. 重新安装平台
pio platform uninstall ststm32
pio platform install ststm32

# 3. 检查网络连接
```

## 优化配置

### 1. 提高构建速度
```ini
# platformio.ini 配置优化
[env:bluepill_f103c8]
platform = ststm32
board = bluepill_f103c8
framework = arduino
build_flags = 
    -Os  # 优化代码大小
    -flto  # 链接时优化
```

### 2. 启用调试支持
```ini
# 添加调试配置
[env:bluepill_f103c8]
debug_tool = stlink
debug_port = COM3
upload_protocol = stlink
```

### 3. 自定义构建脚本
```python
# extra_script.py
Import("env")

# 构建前执行
def before_build(source, target, env):
    print("开始构建STM32项目")

# 构建后执行  
def after_build(source, target, env):
    print(f"构建完成，大小: {env.GetProjectOption('upload_size')}")

env.AddPreAction("buildprog", before_build)
env.AddPostAction("buildprog", after_build)
```

## 自动化脚本

### Windows PowerShell脚本
```powershell
# setup-claude.ps1
Write-Host "正在设置Claude Code开发环境..." -ForegroundColor Green

# 检查Node.js
if (!(Get-Command node -ErrorAction SilentlyContinue)) {
    Write-Host "安装Node.js..." -ForegroundColor Yellow
    winget install OpenJS.NodeJS
}

# 检查Python
if (!(Get-Command python -ErrorAction SilentlyContinue)) {
    Write-Host "安装Python..." -ForegroundColor Yellow
    winget install Python.Python.3.11
}

# 安装MCP服务器
Write-Host "安装MCP服务器..." -ForegroundColor Cyan
npm install -g @modelcontextprotocol/server-filesystem
npm install -g embedded-mcp-server
npm install -g @modelcontextprotocol/server-github

# 安装PlatformIO
Write-Host "安装PlatformIO..." -ForegroundColor Cyan
pip install platformio

Write-Host "环境设置完成!" -ForegroundColor Green
```

### Bash脚本 (macOS/Linux)
```bash
#!/bin/bash
# setup-claude.sh

echo "正在设置Claude Code开发环境..."

# 检查Node.js
if ! command -v node &> /dev/null; then
    echo "安装Node.js..."
    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt-get install -y nodejs
fi

# 检查Python
if ! command -v python3 &> /dev/null; then
    echo "安装Python..."
    sudo apt-get install -y python3 python3-pip
fi

# 安装MCP服务器
echo "安装MCP服务器..."
sudo npm install -g @modelcontextprotocol/server-filesystem
sudo npm install -g embedded-mcp-server
sudo npm install -g @modelcontextprotocol/server-github

# 安装PlatformIO
echo "安装PlatformIO..."
pip3 install platformio

echo "环境设置完成!"
```

## 最佳实践

### 1. 版本控制
```bash
# 使用Git管理配置
git init
git add .
git commit -m "初始配置: Claude Code开发环境"

# 创建.gitignore
echo ".pio/
.vscode/
build/
.DS_Store
*.local" > .gitignore
```

### 2. 文档记录
```markdown
# 项目配置记录

## 环境信息
- **日期**: 2026-04-13
- **Claude Code版本**: 1.0.0
- **PlatformIO版本**: 6.1.0
- **Node.js版本**: 18.15.0
- **Python版本**: 3.11.0

## MCP服务器配置
- filesystem: ✓ 已连接
- embedded: ✓ 已连接
- platformio: ✓ 已连接
- github: ✓ 已连接

## 测试结果
- [x] 项目创建成功
- [x] 构建成功
- [x] 上传成功
- [x] 硬件测试通过
```

### 3. 定期维护
```bash
# 每月更新一次
npm update -g
pip install --upgrade platformio
claude update

# 清理缓存
pio system prune
npm cache clean --force
```

## 结语

正确的配置是高效使用Claude Code进行嵌入式开发的基础。通过本文档的指导，您可以快速搭建完整的开发环境，并开始使用Claude Code进行STM32项目开发。

如果在配置过程中遇到问题，请参考常见问题部分或查阅相关文档。

---
**配置版本**: 1.0.0  
**最后更新**: 2026年4月13日  
**适用平台**: Windows, macOS, Linux  
**目标硬件**: STM32系列开发板