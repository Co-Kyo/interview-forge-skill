# 环境检查与项目部署

**本文件由 SKILL.md 引用，每次 Skill 启动时必须先执行。**

---

## 平台检测（最先执行）

通过 `execute_command` 执行以下检测，确定当前平台并设置对应的命令模板：

1. **检测操作系统**：
   ```bash
   uname -s 2>/dev/null || echo "WINDOWS"
   ```
   - 输出 `Darwin` → macOS
   - 输出 `Linux` → Linux
   - 输出 `MINGW*` 或 `MSYS*` 或 `WINDOWS` → Windows

2. **检测 Git**：
   ```bash
   git --version
   ```
   - 成功 → Git 可用
   - 失败 → **阻断**：告知用户安装 Git（https://git-scm.com/downloads），不继续执行

3. **检测 Node.js ≥ 20**：
   - **macOS / Linux**：`node --version`（需 ≥ 20）
   - **Windows**：`& 'C:\Program Files\Git\bin\bash.exe' -c "source ~/.nvm/nvm.sh && nvm use 20 && node --version"`
   - 成功且版本 ≥ 20 → Node 可用
   - 有 Node 但版本 < 20 → 尝试通过 nvm 安装 20：
     - **macOS / Linux**：`nvm install 20 && nvm use 20`
     - **Windows**：`& 'C:\Program Files\Git\bin\bash.exe' -c "source ~/.nvm/nvm.sh && nvm install 20 && nvm use 20"`
   - 无 Node → **阻断**：告知用户安装 Node.js ≥ 20（推荐 nvm：https://github.com/nvm-sh/nvm 或 Windows 版 https://github.com/coreybutler/nvm-windows），不继续执行

4. **设置平台变量**：检测完成后，为后续所有命令确定执行方式：

   | 项目 | macOS / Linux | Windows |
   |------|--------------|---------|
   | Shell | 直接执行 | `& 'C:\Program Files\Git\bin\bash.exe' -c "..."` |
   | nvm 激活 | `source ~/.nvm/nvm.sh && nvm use 20` | `source ~/.nvm/nvm.sh && nvm use 20`（在 Git Bash 内） |
   | 路径分隔符 | `/` | `/`（Git Bash 内统一用 `/`） |
   | 杀端口进程 | `lsof -ti:{PORT} \| xargs kill -9` | `netstat -ano \| grep :{PORT} \| head -1 \| awk '{print $5}' \| xargs taskkill /PID /F` |

   > **下文中的 `{SHELL_WRAP}` 表示上述平台对应的命令执行方式**。macOS/Linux 直接在 shell 中执行，Windows 需用 Git Bash 包裹。

---

## 项目部署检查

1. **检查 `{SKILL_DIR}/Tools/interview-forge-public/package.json` 是否存在**
2. **存在** → 项目已部署，检查 `node_modules` 是否存在，不存在则执行 `npm install` → 环境就绪
3. **不存在** → 执行自动 clone：

### 自动 clone 流程

**macOS / Linux**：
```bash
cd "{SKILL_DIR}/Tools" && git clone https://github.com/Co-Kyo/interview-forge-public.git && cd interview-forge-public && npm install
```

**Windows**：
```bash
& 'C:\Program Files\Git\bin\bash.exe' -c "source ~/.nvm/nvm.sh && nvm use 20 && cd '{SKILL_DIR}/Tools' && git clone https://github.com/Co-Kyo/interview-forge-public.git && cd interview-forge-public && npm install"
```

> **注意**：前端项目 clone 到 `{SKILL_DIR}/Tools/interview-forge-public/` 子目录，与 Skill 定义文件（SKILL.md/references/Tools/scripts）隔离，互不干扰。

---

## Node.js 版本

InterviewForge 需要 Node.js ≥ 20（`marked@18` 要求）。如平台检测中已确认版本 ≥ 20，此处跳过。

如需安装 Node 20：

**macOS / Linux**：
```bash
source ~/.nvm/nvm.sh && nvm install 20 && nvm use 20
```

**Windows**：
```bash
& 'C:\Program Files\Git\bin\bash.exe' -c "source ~/.nvm/nvm.sh && nvm install 20 && nvm use 20"
```
