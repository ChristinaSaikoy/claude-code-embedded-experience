# Claude Code 嵌入式开发经验分享

> **状态：历史记录 / 不再作为当前作品集项目维护。** 该仓库保留用于记录早期 AI 辅助 STM32 开发经验；我当前的公开作品集主线为 Embedded AI、FPGA 与 Linux Systems。

![Claude Code](https://img.shields.io/badge/Claude-Code-blue)
![STM32](https://img.shields.io/badge/STM32-Embedded-green)
![MCP](https://img.shields.io/badge/MCP-Servers-orange)

这个仓库记录了我早期使用 Claude Code 进行 STM32 嵌入式开发时的经验、技巧和实践，重点包括 MCP 工具链、PlatformIO、STM32 集成与调试。

## 📋 目录
- [项目背景](#项目背景)
- [技术栈](#技术栈)
- [核心经验](#核心经验)
- [MCP服务器使用](#mcp服务器使用)
- [智能车项目](#智能车项目)
- [调试技巧](#调试技巧)
- [常见问题](#常见问题)
- [最佳实践](#最佳实践)

## 🎯 项目背景

早期实践包括：
1. **STM32 LED控制项目** - 验证 MCP 服务器硬件控制流程
2. **智能车集成项目** - 将多个 STM32 模块整合为统一系统
3. **硬件测试项目** - 使用 PlatformIO MCP 进行构建、上传和验证

## 🛠️ 技术栈

| 技术 | 用途 | 状态 |
|------|------|------|
| **Claude Code** | AI辅助开发 | ✅ 已验证 |
| **MCP服务器** | 工具链集成 | ✅ 已验证 |
| **PlatformIO** | STM32项目管理 | ✅ 已验证 |
| **STM32F103C8T6** | 目标硬件 | ✅ 已验证 |
| **Keil MDK** | 传统开发环境 | ✅ 已验证 |

## 💡 核心经验

### 1. MCP服务器分工明确
- **platformio MCP**: 项目管理、构建、上传
- **embedded MCP**: 嵌入式开发知识
- **filesystem MCP**: 文件操作
- **github MCP**: 版本控制

### 2. 硬件开发要点
- **LED极性**: STM32 Blue Pill 的 PC13 LED 是低电平点亮
- **资源冲突**: 提前规划 GPIO 引脚分配
- **非阻塞架构**: 使用 SysTick 替代长时间阻塞 Delay

### 3. AI 辅助开发的边界
- 生成代码仍需结合原理图、数据手册和实测结果验证
- 自动化工具不能替代硬件接口、时序和资源冲突分析
- 调试记录应以可复现现象和测试证据为准

## 🔧 MCP服务器使用

```bash
claude mcp list
```

### PlatformIO项目创建

```bash
mkdir stm32_project
cd stm32_project
pio project init --board bluepill_f103c8
pio run
pio run -t upload
```

## 🚗 智能车项目

```text
智能车/模板/Integrated_Project/
├── Core/
├── Hardware/
├── User/
└── Docs/
```

当时主要处理的问题包括 GPIO 冲突、阻塞调用、中断整合和工程兼容性。

## 🐛 调试要点

- 先验证供电、下载链路和最小外设
- 使用串口和状态 LED 建立可观察性
- 对 GPIO、定时器、中断等共享资源建立统一分配表
- 集成前先分别验证独立模块

## 📚 相关资源

- [Claude Code Documentation](https://docs.anthropic.com/claude/code)
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [PlatformIO Documentation](https://docs.platformio.org/)

## 📄 许可证

本项目采用 MIT 许可证，见 [LICENSE](LICENSE)。

---

**最后更新**: 2026年9月17日  
**状态**: Historical / retained for reference
