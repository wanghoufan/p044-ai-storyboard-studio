# AI 图文短剧分镜生成器

> 一条创意进去，出来五个文字镜头和五张横屏画面；短视频开工前，先把分镜摆好。

[English](README.en.md)

<p>
  <img src="docs/screenshot-home.png" width="220" alt="首页空态：短剧主题、剧本与风格偏好输入">
  <img src="docs/qa/implementation-storyboard-landscape-1440x1024.png" width="760" alt="生成中的分镜结果：左侧创作输入，右侧文字镜头与 16:9 图片">
</p>

<sub>界面取自已运行的线上版本。首页为窄屏空态；分镜结果图为桌面宽度。</sub>

## 它解决什么问题

写短视频之前，得先想清楚每个镜头拍什么、人物做什么、台词说什么、画面什么风格。直接拿剧本开拍，常常拍到一半发现镜头接不上。

这个工具把主题和剧本交给模型：先生成 5 个结构化文字镜头（景别、画面描述、人物动作、台词 / 旁白、画面提示词），再按每个镜头的提示词生成一张 16:9 横屏图。得到的是一份可以直接拿去拍摄或生成视频的分镜稿。

线上地址：<https://ai-storyboard-studio-zeta.vercel.app>

## 核心能力

- **流式文字镜头**：服务端调用百炼 `qwen3.7-plus`，按流式响应逐条显示 5 个镜头，不必等全部生成完。
- **五张横屏画面**：随后调用 `qwen-image-2.0-pro-2026-06-22`，为每个镜头生成一张图，默认尺寸 `2688*1536`（16:9）。
- **结构化镜头字段**：每个镜头包含 `shotNumber`、`visualDescription`、`characterAction`、`dialogueOrNarration`、`shotType`、`visualPrompt`。
- **创作输入**：短剧主题（≤ 80 字）、剧本（≤ 20,000 字）、风格偏好（≤ 400 字）；另有预置风格标签可快速选择。
- **自带 Key（BYOK）**：在页面填入自己的百炼北京地域 API Key，保存在浏览器 `localStorage`，每次请求通过 `x-dashscope-api-key` 请求头带给服务端；可随时清除。
- **导出**：复制全部分镜文字；单张或全部下载图片。下载走服务端签发的凭证，凭证有效期 24 小时。
- **限速与止损**：图片按百炼 2 RPM 的限制逐张排队；生成过程中可随时停止，已完成的内容会保留。

## 快速开始

线上使用：打开 <https://ai-storyboard-studio-zeta.vercel.app>，在「百炼 API Key」里填入自己的 Key 并保存，填好主题与剧本，点生成。费用直接记在你自己的百炼账号上。

本地运行：

```bash
npm install
npm run dev
```

打开终端输出的本地地址即可（Vite 开发服务同时提供 `/api/storyboards`，不需要另起后端）。

本地也可以用环境变量兜底，便于不带浏览器 Key 调试：

```bash
cp .env.example .env
```

在 `.env` 里填写 `DASHSCOPE_API_KEY`。可选变量：

- `DASHSCOPE_BASE_URL`：默认北京地域共享域名 `https://dashscope.aliyuncs.com/compatible-mode/v1`。
- `DASHSCOPE_MODEL`：默认 `qwen3.7-plus`。
- `DASHSCOPE_IMAGE_ENDPOINT`、`DASHSCOPE_IMAGE_MODEL`、`DASHSCOPE_IMAGE_SIZE`：图片模型接口、模型与尺寸，默认 `2688*1536`。

构建与测试：

```bash
npm run build
npm test
```

`npm test` 运行 17 项测试，覆盖输入校验、流式 NDJSON 解析、图片提示词与原生接口参数、下载签名、防篡改校验、复制格式、BYOK 请求头链路（含 Key 不回显断言）与 Key 本地存储。

## 数据与隐私

- API Key 只存在当前浏览器的 `localStorage`；服务端只在本次请求内使用它，不落盘、不写日志、不回显在响应里。
- 本站不持有用户密钥，也不替用户付费；生成的费用直接记在用户自己的百炼账号上。
- 页面没有登录、额度或费用确认，也不在服务端保存生成结果；刷新或关闭页面后，需要重新生成。
- `.env` 已在 `.gitignore` 中，不应提交；线上部署的环境变量不会随源码或构建产物发布。
- 下载凭证由服务端用 HMAC-SHA256 签名，有效期 24 小时；图片本体来自百炼的临时链接。

## 已知限制

- 图片模型按张计费，一次运行默认生成 5 张；受百炼 2 RPM 限制，图片是一张一张排队生成的，整套跑完需要等待。
- 百炼返回的图片链接约 24 小时后失效，请在生成后的会话内及时下载。
- 没有站内登录、使用额度或费用确认；拿到链接并填入自己 Key 的人都能使用生成能力，费用由使用者承担。
- 图片下载接口用服务端配置的 `DASHSCOPE_API_KEY` 校验下载凭证。仅靠浏览器 BYOK、服务端未配置该变量的部署，下载能力是否可用在仓库内未验证。
- `worker/`、`scripts/prepare-sites-build.mjs`、`.openai/hosting.json` 是旧 ChatGPT Sites 通道的遗留，不参与默认构建；需要时用 `npm run build:sites && npm run test:sites` 单独验证。
- 界面与文案只有简体中文。
