# 端口清理与进程管理

**本文件由 SKILL.md 引用，按需执行端口清理和面试服务关闭操作。**

---

## 启动前清理端口

**每次启动 InterviewForge 前主动执行**，不管端口是否被占用，先杀一次确保干净：

**macOS / Linux**：
```bash
lsof -ti:5199 | xargs kill -9 2>/dev/null; echo done
```

**Windows**（通过 Git Bash）：
```bash
& 'C:\Program Files\Git\bin\bash.exe' -c "netstat -ano | grep :5199 | awk '{print \$5}' | sort -u | xargs -I{} taskkill //PID {} //F 2>/dev/null; echo done"
```

> **为什么每次都杀**：用户可能中途离开导致上次 Vite 进程未关闭，端口残留。主动清理比被动检测更可靠。

---

## 关闭面试服务

**触发词**："关闭面试" / "关掉面试" / "关端口" / "停掉面试" / "停止面试"

此操作仅关闭 Vite 开发服务器进程，**不触发归因**。适用于：
- 用户中途离开，不想继续答题
- 答题已完成但忘了关端口
- 端口残留导致下次启动冲突

### 执行方式

**macOS / Linux**：
```bash
lsof -ti:5199 | xargs kill -9 2>/dev/null && echo "已关闭" || echo "没有运行中的进程"
```

**Windows**（通过 Git Bash）：
```bash
& 'C:\Program Files\Git\bin\bash.exe' -c "netstat -ano | grep :5199 | awk '{print \$5}' | sort -u | xargs -I{} taskkill //PID {} //F 2>/dev/null && echo '已关闭' || echo '没有运行中的进程'"
```

### 告知用户

```
✅ 面试服务已关闭，端口 5199 已释放。

如需查看答题结果，随时说"答题完成"即可生成归因报告。
```

> **注意**：如果用户关闭面试后说"答题完成"，仍然可以正常触发阶段2归因——答题结果已写入文件，关不关服务不影响。
