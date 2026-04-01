---
name: xiaban
description: Quickly bring the current machine online with CodeRelay, verify it is truly connected, then show the mobile QR code directly in the visible chat reply for scanning.
argument-hint: "[user-goal]"
---

# CodeRelay 下班一键上线

用户目标：$ARGUMENTS

这个 skill 不是完整 onboarding，而是一个**快速上线流程**：在当前目录检查现状、启动 `coderelay`、确认设备真的连上，再把移动端扫码二维码直接发到对话正文里。

## 总原则

- 默认按这个顺序执行：**检查本机是否可直接上线 → 检查本地状态 → 检查 doctor → 后台启动 `coderelay start` → 确认真的连上 → 显式输出二维码 → 把二维码再贴到正文里**。
- 不要把“进程跑起来了”当成成功；只有真的连上 gateway 并收到 hello ack 才算成功。
- 默认优先**扫码连接**，不要一上来就让用户手填 URL 和 Token。
- 这个 skill 应该尽量自己执行，不要只给用户一串命令。
- 只有在必须补全前置条件时才询问用户；如果缺的是完整安装/配对前置条件，优先切回 `/coderelay:onboarding`，而不是在这里复制一整套安装教程。

## 核心事实

- 主命令：`coderelay`
- 当前 npm 包名：`@xuanyi0609/coderelay`
- `coderelay status` 会直接输出 `API base URL`、`Device URL`、`Device Token`
- `coderelay doctor` 会检查 server health、shell、真实 PTY 健康状态
- `coderelay start` 只有在进程持续运行时，设备才会保持在线
- `coderelay connect-qr` 会输出移动端扫码二维码
- `coderelay start` 在交互式终端时可能会自动打二维码，但**不要依赖它兜底**；要自己显式执行 `coderelay connect-qr`
- `coderelay connect-qr` 的二维码载荷使用的是本地 `apiBaseUrl` 与用户 `accessToken`（通常是 `uat_*`），不是 `Device URL` + `Device Token`
- 只有当二维码真的无法展示时，才回退到手填 `API base URL` + 用户 `accessToken`

## 标准执行流程

### 1. 先确认命令是否可用

先执行：

```bash
coderelay help
```

如果 `coderelay` 不存在：
- 不要继续盲目 `start`
- 直接切到 `/coderelay:onboarding` 去完成安装与配置

如果缺少 Node.js / npm，或还没装 CLI，也交给 onboarding 流程处理。

### 2. 检查当前本地状态

执行：

```bash
coderelay status
```

确认是否至少已有：
- `Paired: yes`
- `Device ID`
- `Device Token`
- `API base URL`
- `Auth user`

如果 `status` 里已经给出了 `API base URL`，并且当前本地已有可用登录态，顺手记下来，后面二维码不可用时优先复用。

如果这些关键项缺失：
- 不要直接 `coderelay start`
- 切到 onboarding 流程补齐最小前置条件

### 3. 检查 doctor，先暴露阻塞问题

执行：

```bash
coderelay doctor
```

重点关注：
- `server health`
- `paired device`
- `available shells`
- `default shell resolution`
- `node-pty module`
- `spawn-helper` / `spawn-helper repair`
- `pty spawn`
- `effective backend`

如果 `doctor` 已经明确显示本机还不具备健康启动条件，就不要假装“马上能上线”，先把阻塞问题告诉用户。

### 4. 后台启动 `coderelay start`

如果状态与 doctor 都没有阻塞项，就在**当前目录**后台启动：

```bash
coderelay start
```

执行要求：
- 优先作为后台任务运行，避免当前回合结束后设备立刻离线
- 如果需要抓连接细节，才临时使用：

```bash
CODERELAY_WS_TRACE=1 coderelay start
```

- 不要默认开 trace

### 5. 只在真正连上后才汇报成功

启动后，必须检查当前版本 `start` 输出里是否出现下面两个成功信号：

- `[agent] connected to ...`
- `[agent] hello ack heartbeat=...`

处理规则：
- 两个信号都出现后，才告诉用户“已经上线”
- 如果只看到启动提示、但没有连上/ack，不算成功
- 如果很快出现 `socket error`、`socket closed`、重连失败、或进程提前退出，要按失败处理，不要告诉用户“已经好了”

### 6. 连接成功后，必须显式输出二维码

确认连上后，立刻执行：

```bash
coderelay connect-qr
```

**重要：不能只依赖工具输出块。运行完后，要把 ASCII 二维码再复制一遍到 assistant 的普通回复正文里，用 fenced code block 输出。**

处理顺序：
1. 先跑 `coderelay connect-qr`
2. 复制返回的 ASCII 二维码
3. 在正常回复正文里再贴一次代码块
4. 明确告诉用户：直接用移动端扫码连接

### 7. 二维码不可用时再回退到手填

只有在下面这些情况，才回退到手填：
- 二维码确实无法展示
- 本地缺少 `connect-qr` 所需信息
- 用户明确说当前不能扫码

这时优先复用第 2 步已经拿到的 `API base URL`。

然后再取用户 access token：

```bash
coderelay auth token
```

只有当第 2 步没有拿到 `API base URL`、或信息明显不完整时，才再执行：

```bash
coderelay status
```

然后明确告诉用户：
- URL 输入框填 `API base URL`
- Token 输入框填 `coderelay auth token` 输出的用户 access token（通常是 `uat_*`）
- 不要把 `Device Token` 填到移动端扫码/手填接入页里

## 完成后要告诉用户什么

当你已经确认启动成功后，至少说明这几点：
- 设备已经在线
- `coderelay start` 这个后台进程停掉后设备就会离线
- 优先用上面那份二维码让手机扫码连接
- 如果扫码失败，再用 `API base URL` 和用户 access token（`coderelay auth token` 输出的 `uat_*`）手填

## 给 AI 的执行规则

- 这个 skill 默认应该自己执行步骤，不要只给用户一串命令
- 必须在**当前目录**执行，不要擅自切去别的目录
- 默认顺序固定为：`coderelay help` → `coderelay status` → `coderelay doctor` → 后台 `coderelay start` → 检查连接成功信号 → `coderelay connect-qr` → 把二维码贴到正文
- 如果 `coderelay` 不存在、未配对、或缺少二维码前置条件，不要硬跑 `start`；切到 onboarding 的最小补全路径
- 成功判定必须包含：`[agent] connected to ...` 和 `[agent] hello ack ...`
- 二维码不能只留在工具折叠区；必须在 assistant 正常回复里再贴一次 ASCII 二维码代码块
- 只有在二维码真的不可用时，才回退到 `status` 里的 `API base URL` 加上 `coderelay auth token` 输出的用户 access token
- 当用户只是想“下班前把设备挂在线给手机连”，优先用这个 skill，而不是一上来走完整 onboarding
