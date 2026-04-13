# STM32智能车项目结构示例

## 项目概述
这是一个STM32智能车集成项目的文件结构示例，展示了如何使用Claude Code组织嵌入式项目。

## 项目结构

```
智能车/模板/Integrated_Project/
├── Core/                    # 核心功能模块
│   ├── SysTick_Manager.c    # 系统滴答定时器管理
│   ├── SysTick_Manager.h
│   ├── Key_StateMachine.c   # 按键状态机
│   ├── Key_StateMachine.h
│   ├── BreathingLED.c       # 呼吸灯效果
│   └── BreathingLED.h
├── Hardware/                # 硬件驱动层
│   ├── Motor.c             # 电机控制
│   ├── Motor.h
│   ├── Encoder.c           # 编码器速度测量
│   ├── Encoder.h
│   ├── PID.c              # PID闭环控制
│   ├── PID.h
│   ├── Timer.c            # 定时器配置
│   ├── Timer.h
│   ├── Serial.c           # 串口通信
│   ├── Serial.h
│   ├── OLED.c             # OLED显示屏驱动
│   └── OLED.h
├── User/                   # 用户应用层
│   ├── Integrated_main.c   # 主程序
│   ├── stm32f10x_it.c     # 中断处理程序
│   └── stm32f10x_it.h
└── Docs/                   # 项目文档
    ├── 引脚分配表.md       # GPIO引脚分配
    ├── 系统连接测试指南.md  # 硬件测试指南
    └── 开发日志.md         # 开发过程记录
```

## 核心模块说明

### 1. SysTick_Manager (系统滴答管理器)
```c
// 非阻塞延时函数
void SysTick_Delay(uint32_t ms);

// 定时器管理
typedef struct {
    uint32_t startTime;
    uint32_t interval;
    void (*callback)(void);
} Timer_t;

void Timer_Init(Timer_t* timer, uint32_t interval, void (*callback)(void));
uint8_t Timer_Check(Timer_t* timer);
```

### 2. Key_StateMachine (按键状态机)
```c
// 按键状态定义
typedef enum {
    KEY_IDLE,
    KEY_PRESS_DETECTED,
    KEY_PRESSED,
    KEY_RELEASED
} KeyState_t;

// 按键扫描函数
KeyState_t Key_Scan(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
```

### 3. BreathingLED (呼吸灯)
```c
// 呼吸灯控制
void BreathingLED_Init(void);
void BreathingLED_Update(void);
void BreathingLED_SetEnable(uint8_t enable);
```

## 硬件驱动层

### 1. Motor.c (电机控制)
```c
// 电机初始化
void Motor_Init(void);

// 设置电机速度
void Motor_SetSpeed(int16_t speed);

// 电机使能控制
void Motor_Enable(uint8_t enable);
```

### 2. PID.c (PID控制器)
```c
// PID结构体
typedef struct {
    float Kp, Ki, Kd;
    float integral;
    float prev_error;
    float output_limit;
} PID_Controller_t;

// PID初始化
void PID_Init(PID_Controller_t* pid, float Kp, float Ki, float Kd, float limit);

// PID计算
float PID_Calculate(PID_Controller_t* pid, float setpoint, float measurement);
```

### 3. OLED.c (显示屏驱动)
```c
// OLED初始化
void OLED_Init(void);

// 显示字符串
void OLED_ShowString(uint8_t x, uint8_t y, char* str);

// 显示数字
void OLED_ShowNumber(uint8_t x, uint8_t y, uint32_t num, uint8_t len);

// 清屏
void OLED_Clear(void);
```

## 主程序结构

### Integrated_main.c
```c
#include "stm32f10x.h"
#include "SysTick_Manager.h"
#include "Key_StateMachine.h"
#include "BreathingLED.h"
#include "Motor.h"
#include "PID.h"
#include "Serial.h"
#include "OLED.h"

// 全局变量
static PID_Controller_t speed_pid;
static Timer_t system_timer;
static uint8_t system_mode = 0;

// 系统初始化
void System_Init(void) {
    // 初始化各模块
    SysTick_Init();
    Key_Init();
    BreathingLED_Init();
    Motor_Init();
    PID_Init(&speed_pid, 1.0, 0.1, 0.05, 1000);
    Serial_Init(115200);
    OLED_Init();
    
    // 初始化系统定时器
    Timer_Init(&system_timer, 1000, System_Timer_Callback);
    
    OLED_ShowString(0, 0, "Smart Car Ready");
    Serial_SendString("System Initialized\r\n");
}

// 系统定时器回调
void System_Timer_Callback(void) {
    static uint32_t counter = 0;
    counter++;
    
    // 每秒更新一次显示
    OLED_ShowNumber(0, 2, counter, 5);
    
    // 发送心跳包
    Serial_SendString("System Alive\r\n");
}

// 串口命令解析
void Serial_CommandParser(char* cmd) {
    if (strcmp(cmd, "LED ON") == 0) {
        BreathingLED_SetEnable(1);
        Serial_SendString("LED Enabled\r\n");
    }
    else if (strcmp(cmd, "LED OFF") == 0) {
        BreathingLED_SetEnable(0);
        Serial_SendString("LED Disabled\r\n");
    }
    else if (strncmp(cmd, "MOTOR ", 6) == 0) {
        int speed = atoi(cmd + 6);
        Motor_SetSpeed(speed);
        Serial_SendString("Motor Speed Set\r\n");
    }
    // ... 更多命令
}

// 主循环
int main(void) {
    System_Init();
    
    while (1) {
        // 非阻塞按键扫描
        KeyState_t key_state = Key_Scan(GPIOA, GPIO_Pin_0);
        if (key_state == KEY_PRESSED) {
            system_mode = (system_mode + 1) % 3;
            OLED_ShowNumber(0, 4, system_mode, 1);
        }
        
        // 更新呼吸灯
        BreathingLED_Update();
        
        // 检查系统定时器
        if (Timer_Check(&system_timer)) {
            System_Timer_Callback();
        }
        
        // 处理串口数据
        if (Serial_Available()) {
            char buffer[64];
            Serial_ReadLine(buffer, sizeof(buffer));
            Serial_CommandParser(buffer);
        }
        
        // PID控制任务
        PID_ControlTask();
    }
}
```

## 中断处理程序

### stm32f10x_it.c
```c
#include "stm32f10x_it.h"
#include "SysTick_Manager.h"

// SysTick中断处理
void SysTick_Handler(void) {
    SysTick_Handler_Impl();
}

// 定时器2中断（用于PID控制）
void TIM2_IRQHandler(void) {
    if (TIM_GetITStatus(TIM2, TIM_IT_Update) != RESET) {
        TIM_ClearITPendingBit(TIM2, TIM_IT_Update);
        // PID控制更新
        g_pid_update_flag = 1;
    }
}

// 串口1中断
void USART1_IRQHandler(void) {
    if (USART_GetITStatus(USART1, USART_IT_RXNE) != RESET) {
        uint8_t data = USART_ReceiveData(USART1);
        Serial_RxCallback(data);
    }
}
```

## 开发经验总结

### 1. 模块化设计
- 每个功能独立成模块，便于测试和维护
- 清晰的接口定义，降低模块间耦合
- 统一的错误处理机制

### 2. 非阻塞架构
- 使用SysTick替代Delay()函数
- 状态机设计，避免阻塞等待
- 定时器管理，实现多任务调度

### 3. 资源管理
- 提前规划GPIO引脚分配
- 统一的中断处理机制
- 合理的内存使用规划

### 4. 调试支持
- 串口调试信息输出
- LED状态指示
- 详细的错误码定义

## 使用Claude Code的优势

### 1. 代码生成
- 快速生成硬件驱动代码
- 自动处理寄存器配置
- 生成完整的项目结构

### 2. 错误检测
- 提前发现资源冲突
- 检查代码逻辑错误
- 验证硬件兼容性

### 3. 文档生成
- 自动生成代码注释
- 创建项目文档
- 生成API文档

### 4. 测试支持
- 生成测试用例
- 模拟硬件行为
- 验证代码正确性

## 项目扩展建议

### 1. 添加新功能
- 无线通信模块（蓝牙/WiFi）
- 传感器数据采集（陀螺仪/加速度计）
- 图像识别功能（摄像头）

### 2. 性能优化
- 代码大小优化
- 运行速度优化
- 功耗优化

### 3. 可靠性提升
- 看门狗定时器
- 错误恢复机制
- 数据备份功能

## 结语

这个项目结构展示了如何使用Claude Code高效地开发STM32嵌入式项目。通过模块化设计、非阻塞架构和清晰的代码组织，可以大大提高开发效率和代码质量。

希望这个示例能为您的嵌入式开发项目提供参考！

---
**项目**: STM32智能车集成项目  
**作者**: Roland  
**日期**: 2026年4月13日  
**工具**: Claude Code + PlatformIO + MCP服务器