---
name: onboarding
description: Guide CodeRelay PC CLI install, pairing, startup, diagnosis, and QR-first mobile connection. Use when users ask how to install coderelay with npm/npx, bring a device online, connect from mobile, or troubleshoot offline and connection issues.
argument-hint: "[user-goal]"
---

# CodeRelay 安装 / 启动 / 调试

用户目标：$ARGUMENTS

帮助用户时，始终优先选择**最短可跑通路径**，不要一开始就堆所有命令。先区分两种场景：

1. **终端用户直接使用已发布 npm 包**
2. **开发者在 `pc/` 仓库里调试源码**

## 核心事实

- 当前 npm 包名：`@xuanyi0609/coderelay`
- 默认控制面地址：`http://218.76.30.251:2087`
- 默认设备接入 URL：`ws://218.76.30.251:2087/ws/agent`
- `pair` 在服务端返回 `debugCode` 时会自动复用，所以通常不必手填 `--code`
- `status` 会直接输出 `Device URL` 和 `Device Token`
- `coderelay connect-qr` 会在终端里输出移动端可扫码的连接二维码
- `coderelay start` 在交互式终端且本地已具备连接信息时，会自动输出同一份连接二维码
- 移动端接入时，默认优先走扫码连接；只有扫不了时才回退到手填 `Device URL` + `Device Token`
- `coderelay auth token` 输出的是用户 access token（`uat_*`），**不是**移动端要填的设备 token；不要混用
- 本地状态默认保存在 `~/.coderelay/`，主要文件包括 `config.json`、`device.json`、`auth.json`
- 设备在线的前提是本机 `coderelay start` 进程正在运行

## 先做环境检查

如果用户让你直接操作，先检查本机是否有 Node.js 和 npm：

```bash
node -v
npm -v
```

如果缺少 Node.js：

- Windows：`! winget install OpenJS.NodeJS.LTS`
- macOS：`! brew install node`
- Linux：先确认发行版，再用系统包管理器安装 Node.js LTS

装完后重新执行：

```bash
node -v
npm -v
```

## 最短工作流：终端用户直接使用（优先推荐）

优先用 `npx`，这样用户不用先全局安装：

```bash
npx @xuanyi0609/coderelay help
npx @xuanyi0609/coderelay pair --email <email>
npx @xuanyi0609/coderelay status
npx @xuanyi0609/coderelay start
```

### 解释

1. `help`：确认包可下载、CLI 可执行
2. `pair --email <email>`：向服务端请求验证码并完成设备配对；如果服务端返回 `debugCode`，CLI 会自动使用，不必手填 `--code`
3. `status`：读取本地状态，直接拿到：
   - `Device URL`
   - `Device Token`
   - `Device ID`
   - 当前 API / Gateway 配置
4. `start`：启动本地 Agent，连接服务端；只要这个进程在运行，设备就能保持在线

## 全局安装工作流

如果用户希望以后直接用 `coderelay` 命令：

```bash
npm i -g @xuanyi0609/coderelay
coderelay help
coderelay pair --email <email>
coderelay status
coderelay start
```

适合长期使用，但首次验证仍然优先推荐 `npx`。

## 仓库源码调试工作流

如果用户当前就在 `D:/Code/CodeRelay/pc` 仓库里开发：

```bash
npm install
npm run build
node dist/cli.js help
node dist/cli.js pair --email <email>
node dist/cli.js status
node dist/cli.js doctor
node dist/cli.js start
```

解释：

- `npm install`：安装源码依赖
- `npm run build`：把 TypeScript 编译到 `dist/`
- `node dist/cli.js ...`：直接验证编译产物，而不是依赖全局安装
- `doctor`：快速判断本地状态、shell、server health，以及真实 PTY 是否健康；在 macOS 上会尝试修复 `node-pty` helper 权限问题

## 移动端连接说明

默认优先让用户**扫码连接**，不要一上来就让用户手填 URL 和 Token。

### 推荐路径：直接输出二维码给移动端扫码

在 PC 上完成 `pair` 后，优先执行：

```bash
npx @xuanyi0609/coderelay connect-qr
```

或（全局安装后）：

```bash
coderelay connect-qr
```

如果用户正准备把设备挂在线，也可以直接执行：

```bash
npx @xuanyi0609/coderelay start
```

或：

```bash
coderelay start
```

在交互式终端且本地已有连接信息时，`start` 会自动把二维码输出出来，移动端可直接扫描。

### 扫码不可用时，再回退到手填

如果终端不是 TTY、二维码没显示出来、或用户当前不方便扫码，再执行：

```bash
npx @xuanyi0609/coderelay status
```

或（全局安装后）：

```bash
coderelay status
```

拿到以下两项：

- `Device URL`
- `Device Token`

然后在移动端的设备接入/连接页面中：

- URL 输入框：填写 `Device URL`
- Token 输入框：填写 `Device Token`

如果用的是手填路径，同时确保 PC 端已经运行：

```bash
npx @xuanyi0609/coderelay start
```

或：

```bash
coderelay start
```

如果移动端显示离线，先不要猜参数，按下面顺序排查：

1. `status` 看是否已配对、URL/Token 是否存在
2. `doctor` 看 server health、shell、真实 PTY 健康状态，以及是否修复了 `node-pty` helper
3. 确认 `start` 进程仍在运行；如果 PTY 不健康，`coderelay start` 现在会直接拒绝启动，避免设备以退化模式上线
4. 必要时重新 `pair`

## 完整命令速查

### 安装与运行

```bash
npx @xuanyi0609/coderelay help
npm i -g @xuanyi0609/coderelay
coderelay help
```

### 配对、扫码与启动

```bash
coderelay pair --email <email>
coderelay connect-qr
coderelay start
coderelay status
coderelay status --remote
```

### 诊断与清理

```bash
coderelay doctor
coderelay unpair
```

### 配置查看与修改

```bash
coderelay config get apiBaseUrl
coderelay config get gatewayUrl
coderelay config set apiBaseUrl http://127.0.0.1:3100
coderelay config set gatewayUrl ws://127.0.0.1:3100/ws/agent
```

### 用户 access token

```bash
coderelay auth token
```

仅在需要调用服务端用户态接口时使用。移动端直连设备时，优先使用 `Device Token`，不要把 `auth token` 当作设备 token。

## 调试顺序

当用户说“显示离线”“连不上”“pair 之后不会用”“移动端不知道填什么”时，按这个顺序处理：

1. 运行：

```bash
coderelay status
```

确认：
- `Paired: yes`
- `Device ID` 不为空
- `Device URL` 不为空
- `Device Token` 不为空

2. 运行：

```bash
coderelay doctor
```

关注：
- `server health`
- `paired device`
- `available shells`
- `default shell resolution`
- `node-pty module`
- `spawn-helper` / `spawn-helper repair`
- `pty spawn`
- `effective backend`

3. 启动 agent：

```bash
coderelay start
```

如果你代替用户执行，优先把它作为后台任务运行，并明确告诉用户：**这个进程停掉后设备就会离线**。

4. 需要抓连接细节时，再开 trace：

```bash
CODERELAY_WS_TRACE=1 coderelay start
```

5. 如果本地状态明显脏了，再执行：

```bash
coderelay unpair
coderelay pair --email <email>
```

## 何时需要自定义服务端地址

默认情况下，不要要求用户手动填写网关或 API 地址。只有在用户明确使用其他环境（本地开发 server / 测试环境）时，才改成：

```bash
coderelay pair --api-base-url http://127.0.0.1:3100 --email <email>
coderelay config set apiBaseUrl http://127.0.0.1:3100
coderelay config set gatewayUrl ws://127.0.0.1:3100/ws/agent
```

## 给 AI 的执行规则

- 当用户要“直接装好并跑起来”时，优先自己执行命令，不要只给口头说明
- 第一次验证优先用 `npx`；只有当用户明确要长期使用时再推荐 `npm i -g`
- 每次成功 `pair` 后，优先执行 `connect-qr` 或启动 `start` 让终端直接输出二维码；不要默认只停留在口头说明
- 如果当前终端是交互式 TTY，并且本地已有连接信息，应优先让用户扫描终端输出的二维码
- 只有在二维码无法展示、无法扫码、或用户明确要求手填时，才回退到 `status` 里的 `Device URL` 和 `Device Token`
- 当用户问“移动端填什么”，先回答“优先扫码连接”；若必须手填，再明确填 `status` 里的 `Device URL` 和 `Device Token`
- 当用户问“为什么显示离线”，先检查 `start` 是否仍在运行，而不是先改配置
- 如果用户在源码仓库里，优先用 `npm install`、`npm run build`、`node dist/cli.js ...` 这组命令
- 不要让用户手工拼接 gateway 地址；默认值够用时直接使用默认值
