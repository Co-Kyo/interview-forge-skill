# 前端启动与目录创建

**本文件由 SKILL.md 引用，按其中流程创建会话目录并启动 InterviewForge 前端。**

---

## 创建会话目录

确保会话目录存在：

**macOS / Linux**：
```bash
mkdir -p "{ARCHIVE_PATH}/sessions/$(date +%Y-%m-%d)"
```

**Windows**（通过 Git Bash）：
```bash
& 'C:\Program Files\Git\bin\bash.exe' -c "mkdir -p '{ARCHIVE_PATH}/sessions/$(date +%Y-%m-%d)'"
```

---

## 启动 InterviewForge

**macOS / Linux**：
```bash
cd {SKILL_DIR}/Tools/interview-forge-public && export QUIZ_FILE='{ARCHIVE_PATH}/sessions/{date}/quiz-{sessionId}.json' && export SESSION_ID='{sessionId}' && export RESULT_DIR='{ARCHIVE_PATH}/sessions/{date}' && npx vite --port 5199 --open
```

**Windows**：
```bash
& 'C:\Program Files\Git\bin\bash.exe' -c "source ~/.nvm/nvm.sh && nvm use 20 && cd {SKILL_DIR}/Tools/interview-forge-public && export QUIZ_FILE='{ARCHIVE_PATH}/sessions/{date}/quiz-{sessionId}.json' && export SESSION_ID='{sessionId}' && export RESULT_DIR='{ARCHIVE_PATH}/sessions/{date}' && npx vite --port 5199 --open"
```

> **注意**：
> - 此命令启动 Vite 开发服务器后会立即返回（不会阻塞）。用户在浏览器中答题，答完后 InterviewForge 会 process.exit(0) 并将结果写入文件。
> - 路径中的 `{ARCHIVE_PATH}` 替换为实际档案路径即可，macOS/Linux 用绝对路径，Windows 用 Windows 绝对路径（Git Bash 会自动转换）。
> - Windows 下如果 execute_command 的默认 shell 是 PowerShell，需要用 `& 'C:\Program Files\Git\bin\bash.exe' -c "..."` 包裹，路径加引号防止空格截断。
