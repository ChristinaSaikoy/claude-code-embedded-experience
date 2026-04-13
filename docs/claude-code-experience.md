# Claude Code 使用经验与技巧分享

## 概述
本文档总结了我在使用Claude Code进行嵌入式开发（特别是STM32项目）过程中的经验、技巧和最佳实践。

## 目录
1. [环境配置](#环境配置)
2. [MCP服务器使用](#mcp服务器使用)
3. [STM32开发流程](#stm32开发流程)
4. [智能车项目集成](#智能车项目集成)
5. [调试技巧](#调试技巧)
6. [常见问题解决](#常见问题解决)

## 环境配置

### 1. Rust环境安装
```bash
# 安装Rust
./rustup-init.exe -y
export PATH="/c/Users/Roland/.cargo/bin:$PATH"

# 验证安装
rustc --version
cargo --version
```

### 2. MCP服务器配置
```bash
# 查看已配置的MCP服务器
claude mcp list

# 关键MCP服务器：
# - filesystem: 文件系统操作
# - embedded: 嵌入式开发工具
# - platformio: PlatformIO项目管理
# - github: GitHub操作
```

## MCP服务器使用

### 1. PlatformIO MCP
- **功能**: STM32项目管理、依赖安装、构建、上传
- **使用场景**: 创建STM32项目、编译代码、上传固件
- **示例命令**:
  ```bash
  # 创建项目
  pio project init --board bluepill_f103c8
  
  # 构建项目
  pio run
  
  # 上传固件
  pio run -t upload
  ```

### 2. Embedded MCP
- **功能**: 提供嵌入式开发知识和工具
- **使用场景**: 硬件配置、引脚分配、外设使用

### 3. GitHub MCP
- **功能**: GitHub仓库操作
- **使用场景**: 创建仓库、推送代码、管理项目

## STM32开发流程

### 1. 项目创建
```bash
# 创建STM32项目目录
mkdir stm32_project
cd stm32_project

# 初始化PlatformIO项目
pio project init --board bluepill_f103c8
```

### 2. 代码编写
**关键文件**: `src/main.cpp`
```cpp
#include <Arduino.h>

void setup() {
  // 初始化代码
  pinMode(PC13, OUTPUT);
  digitalWrite(PC13, HIGH);  // 初始状态：LED灭
  Serial.begin(115200);
  Serial.println("STM32项目启动");
}

void loop() {
  // 主循环代码
  digitalWrite(PC13, LOW);   // LED亮
  delay(1000);
  digitalWrite(PC13, HIGH);  // LED灭
  delay(1000);
}
```

### 3. 构建与上传
```bash
# 构建项目
pio run

# 上传到硬件
pio run -t upload
```

### 4. 硬件注意事项
- **LED极性**: STM32 Blue Pill板上的PC13 LED是**低电平点亮**
- **上传协议**: 自动检测使用ST-LINK
- **驱动**: Windows自动识别，无需额外驱动安装

## 智能车项目集成

### 项目背景
将5个独立的STM32项目整合为统一系统，用于智能车单片机考核。

### 技术架构
- **微控制器**: STM32F103C8T6 (Blue Pill)
- **开发环境**: Keil MDK (μVision)
- **库**: STM32 Standard Peripheral Library
- **架构**: 非阻塞状态机基于SysTick

### 集成模块
1. **呼吸灯模块** (BreathingLED.c/h)
2. **系统滴答管理器** (SysTick_Manager.c/h)
3. **按键状态机** (Key_StateMachine.c/h)
4. **LED驱动** (LED.c/h)
5. **主集成程序** (Integrated_main.c)
6. **中断处理程序** (stm32f10x_it.c)

### 解决的问题
1. **GPIO资源冲突**: 重新分配LED引脚(PA1/PA2 → PC13/PC14)
2. **阻塞调用**: 消除所有Delay()，使用SysTick非阻塞延时
3. **中断冲突**: 统一中断处理，避免重复定义
4. **全局变量命名**: 解决命名冲突
5. **Keil兼容性**: 所有C/H文件以空行结尾

## 调试技巧

### 1. 串口调试
```cpp
// 在代码中添加串口输出
Serial.begin(115200);
Serial.println("调试信息");
```

### 2. LED状态指示
```cpp
// 使用LED指示程序状态
digitalWrite(PC13, LOW);   // 程序运行正常
digitalWrite(PC13, HIGH);  // 程序异常
```

### 3. 非阻塞调试
```cpp
// 使用非阻塞方式输出调试信息
static uint32_t lastDebugTime = 0;
if (millis() - lastDebugTime > 1000) {
  Serial.println("每秒输出一次");
  lastDebugTime = millis();
}
```

## 常见问题解决

### 1. LED不亮
1. 检查开发板供电（USB连接）
2. 确认LED在PC13引脚（Blue Pill板载LED）
3. 验证固件是否成功上传（查看上传日志）
4. 检查代码中的引脚定义和电平逻辑

### 2. 构建失败
1. 检查网络连接（PlatformIO需要下载工具链）
2. 验证board名称正确性 (`bluepill_f103c8`)
3. 检查platformio.ini配置

### 3. 上传失败
1. 检查ST-LINK连接
2. 尝试手动指定上传协议: `pio run -t upload --upload-port=stlink`
3. 确认开发板处于可编程模式（有些板需要设置BOOT跳线）

## 最佳实践

### 1. 代码组织
- 模块化设计，每个功能独立成文件
- 清晰的函数命名和注释
- 统一的代码风格

### 2. 版本控制
- 使用Git进行版本管理
- 定期提交，清晰的提交信息
- 使用分支进行功能开发

### 3. 文档记录
- 记录硬件连接图
- 记录引脚分配表
- 记录调试过程和解决方案

### 4. 测试策略
- 单元测试每个模块
- 集成测试整个系统
- 硬件在环测试

## 经验总结

### 1. MCP的实际价值
- 通过标准化协议整合开发工具链
- 提供统一的硬件访问接口
- 实现从代码到硬件运行的完整自动化

### 2. 硬件开发要点
- 理解硬件特性（如LED极性）
- 正确处理资源冲突
- 设计非阻塞架构

### 3. Claude Code优势
- 智能代码生成和修改
- 自动化测试和验证
- 完整的开发流程支持

## 未来计划

### 1. 扩展应用
- 添加更多外设：按钮、传感器、显示屏
- 实现复杂逻辑：状态机、通信协议、数据处理
- 开发自定义MCP服务器

### 2. 团队协作
- 通过MCP标准化开发流程
- 建立团队开发规范
- 创建项目模板

### 3. 知识分享
- 撰写更多技术文档
- 录制教学视频
- 参与开源社区

## 结语

Claude Code结合MCP服务器为嵌入式开发带来了革命性的变化。通过标准化的工具链和智能化的代码生成，大大提高了开发效率和代码质量。希望这些经验分享能帮助更多开发者更好地使用Claude Code进行嵌入式开发。

---

**作者**: Roland  
**日期**: 2026年4月13日  
**项目**: STM32智能车集成项目  
**工具**: Claude Code + MCP服务器