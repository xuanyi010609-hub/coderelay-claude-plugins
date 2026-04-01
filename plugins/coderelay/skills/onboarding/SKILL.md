---
name: onboarding
description: Fully set up CodeRelay on the user's machine from install to configuration to startup. Use when the user wants CodeRelay ready to use, wants mobile connection set up, or needs end-to-end onboarding. Prefer doing the steps yourself and ask only for required missing info such as email.
argument-hint: "[user-goal]"
---

# CodeRelay 一键安装 / 配置 / 启动

用户目标：$ARGUMENTS

这个 skill 的目标不是只给说明，而是**尽量由 agent 自己完成从检查、安装/更新、配置、启动到连接说明的全过程**。除非遇到必须由用户提供的信息或必须用户亲自完成的交互，否则不要把步骤甩回给用户。

## 总原则

- 默认按这个顺序执行：**检查是否已安装 → 首次安装或更新 → 检查配置 → 需要时补配置 → 启动 → 告知后续使用方式**。
- 只有在以下情况才询问用户：
  - 缺少完成 `pair` 所必须的邮箱
  - 需要用户完成邮箱验证码/登录这类外部身份验证
  - 缺少 Node.js 且安装动作需要用户确认或手动授权
  - 存在用户必须自己决定的环境信息（例如明确指定测试环境地址）
- 如果本机已经配置好，就跳过配置步骤，不要重复让用户输入。
- **首次使用优先 `npm install -g @xuanyi0609/coderelay`，不要默认走 `npx`。**
- 完成安装后，后续统一使用 `coderelay ...` 命令，而不是继续混用 `npx`。
- 如果已经装过 `coderelay`，优先检查是否可直接使用；必要时再升级。
- 移动端连接默认优先**扫码连接**，并尽量主动把二维码输出出来。

## 核心事实

- 当前 npm 包名：`@xuanyi0609/coderelay`
- 安装后主命令：`coderelay`
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

## 标准执行流程

### 1. 先检查运行环境

如果用户希望你直接操作，先自己检查：

```bash
node -v
npm -v
coderelay help
```

解释：
- `node -v` / `npm -v`：确认 Node.js 与 npm 是否存在
- `coderelay help`：确认是否已经安装过 CLI

如果 `coderelay` 不存在，但 Node/npm 已存在，则直接进入安装步骤。

如果缺少 Node.js：
- Windows：建议用户执行 `! winget install OpenJS.NodeJS.LTS`
- macOS：建议用户执行 `! brew install node`
- Linux：根据发行版选择系统包管理器安装 Node.js LTS

装完后重新检查：

```bash
node -v
npm -v
coderelay help
```

### 2. 首次安装或更新

如果还没安装 `coderelay`，优先由 agent 直接执行：

```bash
npm install -g @xuanyi0609/coderelay
coderelay help
```

如果已经安装但明显需要更新，也可执行：

```bash
npm install -g @xuanyi0609/coderelay
coderelay help
```

不要把首次安装主路径写成 `npx`。`npx` 可以作为临时兜底，但默认产品路径应是“先全局安装，再长期用 `coderelay`”。

### 3. 检查是否已经配置完成

安装后优先执行：

```bash
coderelay status
```

目标是判断：
- 是否已经配对
- 是否已有 `Device ID`
- 是否已有 `Device URL`
- 是否已有 `Device Token`

如果这些信息已经齐全，说明用户大概率已经完成配置，可以跳过配对流程，直接进入启动。

### 4. 未配置时再补配置

如果 `status` 显示未配对或缺少关键连接信息，再执行配置流程。

优先使用默认服务端，不要先让用户手工填 URL。

如果缺少邮箱，**这是必须向用户询问的最小信息**。询问应尽量直接，例如：
- “需要一个用于配对的邮箱地址，我来继续完成配置。你要用哪个邮箱？”

拿到邮箱后执行：

```bash
coderelay pair --email <email>
```

然后再次执行：

```bash
coderelay status
```

并确认至少拿到：
- `Device ID`
- `Device URL`
- `Device Token`

如果用户明确是在本地开发 / 测试环境中使用，才改成自定义服务端地址：

```bash
coderelay pair --api-base-url http://127.0.0.1:3100 --email <email>
```

只有在用户明确要求测试环境时，才继续让用户或 agent 写入：

```bash
coderelay config set apiBaseUrl http://127.0.0.1:3100
coderelay config set gatewayUrl ws://127.0.0.1:3100/ws/agent
```

### 5. 启动 agent

配置完成后，由 agent 直接启动：

```bash
coderelay start
```

如果你代替用户执行，优先把它作为后台任务运行，并明确告诉用户：
- **这个进程停掉后设备就会离线**

必要时可用：

```bash
CODERELAY_WS_TRACE=1 coderelay start
```

但只在抓连接细节时再开，不要默认开启。

### 6. 优先完成扫码连接

默认优先让用户**扫码连接**，不要一上来就让用户手填 URL 和 Token。

优先顺序如下：

1. 如果 `coderelay start` 已在交互式终端里自动输出二维码，直接提示用户扫码
2. 如果没有自动显示，但本地连接信息已经齐全，主动执行：

```bash
coderelay connect-qr
```

3. 如果当前环境不是 TTY、二维码没显示出来、或用户当前不方便扫码，再回退到：

```bash
coderelay status
```

然后把以下两项告诉用户：
- `Device URL`
- `Device Token`

并明确说明：
- URL 输入框填 `Device URL`
- Token 输入框填 `Device Token`

## 完成后必须告诉用户什么

当安装/配置/启动完成后，不要只说“好了”。要明确告诉用户后续如何使用：

### 后续常用命令

```bash
coderelay start
coderelay status
coderelay connect-qr
coderelay doctor
coderelay unpair
```

### 用户应理解的要点

- 以后日常使用直接运行 `coderelay start`
- 想看当前设备信息运行 `coderelay status`
- 想给手机扫码连接运行 `coderelay connect-qr`
- 如果设备显示异常或离线，先运行 `coderelay doctor`
- 如果 `start` 进程停了，设备就会离线

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

3. 启动或重启 agent：

```bash
coderelay start
```

4. 如果需要移动端重新接入，优先再输出二维码：

```bash
coderelay connect-qr
```

5. 如果本地状态明显脏了，再执行：

```bash
coderelay unpair
coderelay pair --email <email>
```

## 给 AI 的执行规则

- 这个 skill 默认应该**自己执行步骤**，不要只给用户一串命令让用户自己跑
- 默认流程是：检查 → 安装/更新 → 检查配置 → 必要时配对 → 启动 → 输出二维码/连接方式 → 告知后续使用方式
- 只有在缺少邮箱、验证码、登录确认、Node 安装授权等必要外部条件时才询问用户
- 第一次使用默认优先 `npm install -g @xuanyi0609/coderelay`
- 安装完成后统一使用 `coderelay` 命令，不要默认继续用 `npx`
- 每次成功 `pair` 后，优先执行 `connect-qr` 或启动 `start` 让终端直接输出二维码；不要默认只停留在口头说明
- 如果当前终端是交互式 TTY，并且本地已有连接信息，应优先让用户扫描终端输出的二维码
- 只有在二维码无法展示、无法扫码、或用户明确要求手填时，才回退到 `status` 里的 `Device URL` 和 `Device Token`
- 当用户问“移动端填什么”，先回答“优先扫码连接”；若必须手填，再明确填 `status` 里的 `Device URL` 和 `Device Token`
- 当用户问“为什么显示离线”，先检查 `start` 是否仍在运行，而不是先改配置
- 如果用户明确在源码仓库里调试，才切到 `npm install`、`npm run build`、`node dist/cli.js ...` 这组命令
- 不要让用户手工拼接 gateway 地址；默认值够用时直接使用默认值
