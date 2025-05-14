---
title: Trae
description: 博客配置
date: 2025-04-20
slug: Trae
image: pawel-czerwinski-8uZPynIu-rQ-unsplash_hu3425483315149503896.jpg
categories:
  - Trae
---

# Trae 终端乱码

以下是无需修改系统编码的解决方案：

---

### **临时解决方案**

#### 1. **强制 VSCode 终端使用 UTF-8**

- **临时修改当前会话编码**：  
  在 VSCode 终端中执行以下命令，强制切换到 UTF-8 代码页：
  ```powershell
  chcp 65001
  ```
- **验证效果**：  
  重新运行程序，观察乱码是否消失。

#### 2. **排除第三方日志库干扰**

- **问题现象**：  
  日志配置中使用了 `com.yomahub.tlog` 的增强编码器（`AspectLogbackEncoder`），可能未正确处理 UTF-8。
- **临时替换为标准编码器**：  
  修改 `logback-spring.xml`，改用原生的 `PatternLayoutEncoder`：
  ```xml
  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
      <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
          <pattern>${CONSOLE_LOG_PATTERN}</pattern>
          <charset>UTF-8</charset>
      </encoder>
  </appender>
  ```

#### 3. **显式指定 JVM 参数**

- **在 VSCode 启动配置中添加参数**：  
  打开 `.vscode/launch.json`，在 `vmArgs` 中追加：
  ```json
  "vmArgs": "-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8"
  ```
- **完整示例**：
  ```json
  {
    "type": "java",
    "name": "Launch Program",
    "request": "launch",
    "vmArgs": "-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8",
    "mainClass": "top.continew.admin.ContiNewAdminApplication"
  }
  ```

#### 4. **检查资源文件编码**

- **确保所有 `.properties` 文件为 UTF-8**：  
  在 VSCode 中打开文件，点击右下角编码标签（如 `UTF-8`），选择 `Save with Encoding -> UTF-8`。
- **在代码中显式指定资源文件编码**：
  ```java
  ResourceBundle bundle = ResourceBundle.getBundle("messages", Locale.getDefault(),
      new UTF8Control());  // 自定义 UTF-8 解码
  ```

#### 5. **使用其他终端工具验证**

- **临时切换到 Git Bash**：  
  在 VSCode 中，点击终端下拉菜单 → `Select Default Profile` → 选择 `Git Bash`，重新运行程序。

---

### **验证步骤**

1. **重新启动程序**，观察输出是否正常。
2. **测试中文字符**：在代码中添加 `System.out.println("测试中文");`，确认输出无乱码。

---

### **总结**

- **核心问题**：VSCode 终端默认使用 GBK 编码，且第三方日志增强库可能未正确处理 UTF-8。
- **修复步骤**：强制终端编码 → 替换日志编码器 → 显式指定 JVM 参数 → 检查资源文件。
- **零侵入性**：无需修改系统设置，仅通过应用层配置解决问题。
