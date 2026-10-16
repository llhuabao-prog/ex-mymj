# 微信 AI 人格机器人 · 使用教程

> 把真实的微信聊天记录蒸馏成会说话的人格，在微信里用 TA 的口吻陪你聊天。

---

## 目录

- [一、项目简介](#一项目简介)
- [二、环境准备](#二环境准备)
- [三、快速开始（三步走）](#三快速开始三步走)
- [四、聊天记录准备（jsonl 格式）](#四聊天记录准备jsonl-格式)
- [五、大模型配置说明](#五大模型配置说明)
- [六、常用命令一览](#六常用命令一览)
- [七、切换人设 / 切换模型](#七切换人设--切换模型)
- [八、把项目打包发给别人](#八把项目打包发给别人)
- [九、项目结构说明](#九项目结构说明)
- [十、常见问题](#十常见问题)
- [十一、注意事项与隐私](#十一注意事项与隐私)

---

## 一、项目简介

本项目是一个**微信 AI 人格机器人**，核心思路是：

```
真实聊天记录 → 统计口癖/语气/性格 → 大模型生成人设 → 微信扫码登录 → 用 TA 的口吻自动回复
```

机器人收到消息后的处理流程：

```
收到消息 → chat_rules 约束说话方式 → persona 决定语气 → memories 提供真实细节 → 输出回复
```

**特点：**
- 口癖、语气词等"事实"全部来自你聊天记录的客观统计，模型只负责组织语言，**不会凭空编造**
- 只需三步即可跑起来：填 Key → 放聊天记录 → 微信扫码
- 支持任意 OpenAI 兼容的大模型服务商（DeepSeek / OpenAI / Kimi / 通义 / 智谱 / 自定义）
- 纯 Node.js 实现，无需 Python，无需 `npm install`

---

## 二、环境准备

### 1. 安装 Node.js

项目要求 **Node.js ≥ 18**（已内置 `fetch`，无需安装额外依赖）。

- 下载地址：<https://nodejs.org>（建议安装 LTS 版本）
- 安装完成后，打开终端验证：

```bash
node -v
```

能打印出版本号（如 `v20.18.0`）即表示安装成功。

### 2. 解压项目

将 `my-ex-workflow.zip` 解压到任意目录，进入该目录即可。

---

## 三、快速开始（三步走）

> 以 Windows 为例，macOS / Linux 把 `.bat` 换成 `.sh` 即可。

### 第 ① 步：配置大模型 Key

在项目目录下，**双击 `setup.bat`**（或在终端执行 `npm run setup`）。

向导会引导你：

1. **选择服务商**：输入序号选择（1=DeepSeek，2=OpenAI，3=Kimi，4=通义千问，5=智谱 GLM，6=自定义）
2. **粘贴 API Key**：从对应服务商控制台申请
3. **确认 Base URL 和模型名**：一般直接回车使用默认值

配置会自动写入 `bot/.env` 文件，无需手动编辑。

> 如果已有 `.env` 配置，向导会询问是否重新配置。

### 第 ② 步：放入聊天记录并生成人设

向导接着会问你是否要生成人设：

- 输入聊天记录 `.jsonl` 文件的路径（没有可回车跳过，稍后单独生成）
- 输入 TA 的昵称/代号（可选）
- 输入一句话基本情况（可选，如"在一起三年，大学同学"）
- 输入一句话性格标签（可选，如"ENFP 焦虑型 爱撒娇"）

脚本会自动统计语料并生成人设三件套，写入 `exes/<昵称>/` 目录。

**也可以跳过向导，单独执行生成：**

```bash
npm run build -- --file ./聊天记录.jsonl --name 小美
```

### 第 ③ 步：微信扫码启动

**双击 `start.bat`**（或终端执行 `npm start`）。

- 首次启动会在终端打印二维码
- 用**微信扫码登录**
- 登录成功后，机器人就会自动用 TA 的口吻回复消息了

> 如果还没配置 Key，`npm start` 会自动先拉起配置向导。

---

## 四、聊天记录准备（jsonl 格式）

机器人读取的是 `.jsonl` 文件，**每行一条 JSON 消息**，格式如下：

```json
{"time": "2026-07-20 21:03:11", "sender": "小美", "content": "在干嘛呢", "is_me": false}
{"time": "2026-07-20 21:03:40", "sender": "我", "content": "刚下班", "is_me": true}
```

### 字段说明

| 字段 | 说明 | 是否必填 |
|------|------|----------|
| `time` | 消息时间，格式 `YYYY-MM-DD HH:mm:ss`，用于排序 | 可选（建议保留） |
| `sender` | 发送者昵称，你自己的消息建议统一写 `我` | 必填 |
| `content` | 消息内容 | 必填 |
| `is_me` | `true` = 你发的，`false` = 对方发的 | 必填 |

### 如何导出微信聊天记录

微信聊天记录可使用 [WechatExporter](https://github.com/nicktang1990/WechatExporter) 等工具导出为 txt/csv，再整理成上面的 jsonl 格式。

> 项目目录下附带了 `微信聊天导出工具.exe`，可用于导出聊天记录。

### 提升人设质量的建议

- **聊天记录质量决定人设质量**：真实记录越多越像
- 建议优先收集：TA **主动发的**长消息 > **情感类消息** > 日常消息
- 至少几十条以上效果才比较明显，几百到几千条最佳

---

## 五、大模型配置说明

配置文件为 `bot/.env`（由 `npm run setup` 自动生成），核心变量：

| 变量 | 说明 |
|------|------|
| `LLM_API_KEY` | 大模型 API Key（必填） |
| `LLM_BASE_URL` | 接口地址 |
| `LLM_MODEL` | 模型名 |
| `PERSONA_DIR` | 指定加载哪份人设（留空自动选择） |

### 各服务商参考配置

| 服务商 | `LLM_BASE_URL` | `LLM_MODEL` |
|--------|----------------|-------------|
| DeepSeek（推荐） | `https://api.deepseek.com` | `deepseek-chat` |
| OpenAI | `https://api.openai.com/v1` | `gpt-4o-mini` |
| Kimi（月之暗面） | `https://api.moonshot.cn/v1` | `moonshot-v1-8k` |
| 通义千问 | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-plus` |
| 智谱 GLM | `https://open.bigmodel.cn/api/paas/v4` | `glm-4-flash` |

> 任何兼容 OpenAI 接口格式的服务商或中转站都可以使用，选"自定义"填入即可。

---

## 六、常用命令一览

| 命令 | 作用 |
|------|------|
| `npm run setup` | 配置向导（选服务商 / 填 Key / 放聊天记录 / 生成人设） |
| `npm run build -- --file x.jsonl --name 名字` | 只用 jsonl 生成人设（不跑交互） |
| `npm start` | 检查配置与人设，然后启动微信机器人 |
| `npm run bot` | 跳过检查，直接启动机器人 |
| `npm run pack` | 打包成干净副本（不含 Key 和聊天记录），方便发给别人 |

> Windows 用户可直接双击对应的 `.bat` 文件：`setup.bat`、`start.bat`。

---

## 七、切换人设 / 切换模型

### 切换人设

不同的聊天记录会生成到各自的 `exes/<slug>/` 目录下。

- 在 `bot/.env` 中设置 `PERSONA_DIR` 指定要加载哪一份：

```env
PERSONA_DIR=./exes/小美
```

- 留空则自动选择 `exes/` 下的第一份人设。

### 切换模型

修改 `bot/.env` 中的 `LLM_API_KEY`、`LLM_BASE_URL`、`LLM_MODEL` 三个变量即可。

---

## 八、把项目打包发给别人

如果你想把这个项目分享给朋友（让 TA 也能生成自己的人格机器人）：

```bash
npm run pack                       # 生成 dist/ 下的干净目录和 zip
npm run pack -- --with-personas    # 连已生成的人设一起打包
npm run pack -- --name my-bot      # 自定义包名
```

打包后的副本**不包含**：
- `bot/.env`（你的密钥）
- `exes/` 下的真实聊天记录与人设（除非加了 `--with-personas`）
- 运行状态、日志等

对方解压后按包内 `开始使用.txt` 的三步走即可。

---

## 九、项目结构说明

```
my-ex-workflow/
├── package.json              # npm 脚本定义
├── setup.bat / start.bat     # Windows 一键脚本
├── setup.sh / start.sh       # macOS / Linux 一键脚本
├── 开始使用.txt               # 快速上手指引
│
├── scripts/                  # 开箱脚本（纯 Node，无第三方依赖）
│   ├── setup.mjs             #   配置向导
│   ├── build_persona.mjs     #   jsonl → 人设三件套
│   ├── start.mjs             #   检查配置 → 启动机器人
│   ├── pack.mjs              #   打包分发
│   └── lib/                  #   工具库（路径/env/llm/jsonl/语料统计）
│
├── bot/                      # 微信机器人
│   ├── wechat_bot.mjs        #   主程序（扫码登录 + 长轮询 + 调大模型）
│   ├── .env.example          #   配置模板
│   └── .env                  #   你的密钥（不随包分发）
│
├── prompts/                  # 人设生成用的 Prompt 模板
│   ├── persona_analyzer.md   #   性格行为提取
│   ├── persona_builder.md    #   persona.md 生成模板
│   ├── chat_rules.universal.md  # 通用聊天规则（反编造/情绪底线）
│   ├── chat_rules.persona.md #   人设专属口癖段
│   ├── memories_analyzer.md  #   共同记忆提取
│   └── memories_builder.md   #   memories.md 生成模板
│
└── exes/                     # 生成的人设（含真实记录，勿外传）
    └── {昵称}/
        ├── chat_rules.md     #   说话规则
        ├── persona.md        #   性格与表达风格
        ├── memories.md       #   共同记忆
        ├── meta.json         #   语料统计等元数据
        └── knowledge/        #   原始语料归档
```

### 生成的人设三件套

| 文件 | 内容 |
|------|------|
| `chat_rules.md` | 硬规则：怎么说话、别编造、情绪底线 |
| `persona.md` | 性格与表达风格（五层结构） |
| `memories.md` | 共同记忆（可被自然引用） |

---

## 十、常见问题

### Q1：终端没有二维码？

确认终端窗口足够大，日志没有把二维码冲掉；重新执行 `npm start` 即可。

### Q2：回复报错 / 401 Unauthorized？

检查 `bot/.env` 里的 `LLM_API_KEY`、`LLM_BASE_URL`、`LLM_MODEL` 是否与你选的服务商匹配。

### Q3：人设不像 TA？

多喂一些 TA **本人发的**聊天记录，然后重新执行 `npm run build`。真实记录越多越像。

### Q4：想换一个人怎么办？

用新的聊天记录执行 `npm run build -- --file 新记录.jsonl --name 新昵称`，会生成到新的 `exes/<昵称>/` 目录，再修改 `bot/.env` 的 `PERSONA_DIR` 指向新目录即可。

### Q5：机器人不回复？

- 确认微信扫码登录成功
- 确认网络能访问大模型接口
- 查看终端是否有报错信息

### Q6：`npm run build` 生成失败？

确认 `bot/.env` 中的 Key 有效，且能正常调用大模型接口。可稍后重试：`npm run build -- --file 你的.jsonl`。

---

## 十一、注意事项与隐私

### 隐私安全

- 所有聊天记录仅在**本地处理**，不会上传到任何服务器
- 只有调用大模型生成人设和回复时，才会把拼好的上下文发给你自己配置的服务商
- `bot/.env`（密钥）和 `exes/`（真实聊天记录）默认**不随打包分发**
- 不要把 `bot/.env` 和 `exes/` 目录提交到 git 或分享给别人

### 使用建议

- 聊天记录质量决定人设质量，建议收集足够多的真实记录
- 优先收集 TA 主动发的长消息和情感类消息
- 本项目仅供个人学习和情感陪伴使用，请遵守相关法律法规和平台规则

---

## 附：效果示例

> 输入：`ENFP 双子座 焦虑型 爱撒娇 翻旧账 嘴上说不在意其实比谁都在意`

**日常聊天：**

```
你      > 在干嘛呢

机器人  > 刚下班
        > 好累啊今天
        > 你呢 怎么这个点才找我 🥺
```

**惹她生气了：**

```
你      > 不好意思刚才在忙

机器人  > 哦
        > （过了5分钟）
        > 你每次都说在忙
        > 上次也是这样
```

**撒娇：**

```
你      > 想吃什么

机器人  > 随便啊
你      > 那吃面？
机器人  > 不想吃面
你      > 火锅？
机器人  > 嘿嘿你怎么知道的 💕
```

---

*视频教程抖音搜“NOMI 玩转 AI”*
