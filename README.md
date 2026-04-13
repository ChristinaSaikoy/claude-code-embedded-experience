# Claude Code 嵌入式开发经验分享

![Claude Code](https://img.shields.io/badge/Claude-Code-blue)
![STM32](https://img.shields.io/badge/STM32-Embedded-green)
![MCP](https://img.shields.io/badge/MCP-Servers-orange)

这个仓库分享了我使用Claude Code进行STM32嵌入式开发的经验、技巧和最佳实践。特别关注智能车项目的集成开发。

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

作为嵌入式软件开发者，我使用Claude Code完成了以下项目：
1. **STM32 LED控制项目** - 验证MCP服务器硬件控制流程
2. **智能车集成项目** - 将5个独立STM32项目整合为统一系统
3. **硬件测试项目** - 使用PlatformIO MCP进行固件上传和测试

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
- **LED极性**: STM32 Blue Pill的PC13 LED是**低电平点亮**
- **资源冲突**: 提前规划GPIO引脚分配
- **非阻塞架构**: 使用SysTick替代Delay()

### 3. Claude Code优势
- **智能代码生成**: 快速创建硬件驱动代码
- **错误检测**: 提前发现资源冲突
- **自动化测试**: 验证代码正确性

## 🔧 MCP服务器使用

### 环境配置
```bash
# 查看MCP服务器状态
claude mcp list

# 关键服务器输出：
# - platformio: ✓ Connected
# - embedded: ✓ Connected
# - filesystem: ✓ Connected
# - github: ✓ Connected
```

### PlatformIO项目创建
```bash
# 创建STM32项目
mkdir stm32_project
cd stm32_project
pio project init --board bluepill_f103c8

# 构建和上传
pio run
pio run -t upload
```

## 🚗 智能车项目

### 项目结构
```
智能车/模板/Integrated_Project/
├── Core/           # 核心模块
│   ├── SysTick_Manager.c/h
│   ├── Key_StateMachine.c/h
│   └── BreathingLED.c/h
├── Hardware/       # 硬件驱动
│   ├── Motor.c/h
│   ├── Encoder.c/h
│   ├── PID.c/h
│   └── OLED.c/h
├── User/          # 用户代码
│   └── Integrated_main.c
└── Docs/          # 文档
```

### 解决的问题
1. **GPIO冲突**: LED引脚从PA1/PA2改为PC13/PC14
2. **阻塞调用**: 全部改为非阻塞状态机
3. **中断冲突**: 统一中断处理程序
4. **Keil兼容**: 所有文件以空行结尾

## 🐛 调试技巧

### 串口调试
```cpp
void setup() {
  Serial.begin(115200);
  Serial.println("系统启动");
}

void loop() {
  static uint32_t lastTime = 0;
  if (millis() - lastTime > 1000) {
    Serial.println("运行正常");
    lastTime = millis();
  }
}
```

### LED状态指示
- **常亮**: 系统正常
- **闪烁**: 正在处理
- **快速闪烁**: 错误状态

## ❓ 常见问题

### Q1: LED不亮怎么办？
1. 检查USB供电
2. 验证固件上传成功
3. 确认LED极性（STM32是低电平点亮）
4. 检查引脚配置

### Q2: 构建失败怎么办？
1. 检查网络连接
2. 验证board名称
3. 检查platformio.ini配置

### Q3: 上传失败怎么办？
1. 检查ST-LINK连接
2. 尝试指定上传协议
3. 确认BOOT跳线设置

## 📚 最佳实践

### 代码组织
- 模块化设计，功能分离
- 清晰的命名规范
- 详细的注释说明

### 版本控制
- 使用Git管理代码
- 定期提交，信息清晰
- 分支开发，主分支稳定

### 文档记录
- 硬件连接图
- 引脚分配表
- 调试日志

### 测试策略
- 模块单元测试
- 系统集成测试
- 硬件在环测试

## 🎓 学习资源

### 官方文档
- [Claude Code Documentation](https://docs.anthropic.com/claude/code)
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [PlatformIO Documentation](https://docs.platformio.org/)

### 相关项目
- [STM32 Blue Pill Examples](https://github.com/stm32duino/Arduino_Core_STM32)
- [PlatformIO STM32](https://github.com/platformio/platform-ststm32)

## 🤝 贡献

欢迎提交Issue和Pull Request分享您的Claude Code使用经验！

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 👤 作者

**Roland** - 嵌入式软件开发者

- GitHub: [@ChristinaSaikoy](https://github.com/ChristinaSaikoy)
- 项目经验: STM32、嵌入式系统、AI辅助开发

## 🙏 致谢

感谢 Anthropic 团队开发 Claude Code，为嵌入式开发带来革命性的变化！

---

**最后更新**: 2026年4月13日  
**状态**: 持续更新中