# dsh-zh-reasoning

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache--2.0-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/ZiYe4u/dsh-zh-reasoning?style=social)](https://github.com/ZiYe4u/dsh-zh-reasoning)

让 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 的**思考（reasoning）与最终回答**默认使用简体中文的插件。

> **本仓库是 [zhy201810576/dsh-zh-reasoning](https://github.com/zhy201810576/dsh-zh-reasoning) 的非官方衍生版本（unofficial fork）**，
> 与原作者之间不存在隶属、赞助或背书关系；后者又是基于
> [imlishiyuan/deepseek-harness-zh-cn](https://github.com/imlishiyuan/deepseek-harness-zh-cn)（Apache-2.0）改写而来。
> 本仓库在其基础上修复了 DSH 会话格式 v4 的 source 署名问题，并改为零运行时依赖。
> 版权与归属详见文末「版权与归属」一节。

## 功能特性

- 在模型每个会话的第一步（`agent/pre-step`）注入一条 `<system-reminder>` 指令，要求：
  - 思考（reasoning）与最终回答使用简体中文；
  - 计划、工具调用、总结、代码注释同样使用中文；
  - 仅代码、命令、文件路径、变量名、API 名称等保留英文；
  - 除非用户明确要求其他语言，否则规则优先。
- 同时影响模型的**最终回答**与**思维链（reasoning）**。
- 标准插件结构，支持 `dsh plugin` 挂载、GitHub 安装与更新。
- **零运行时依赖**：不打包任何 `node_modules`，`@deepseek-ai/dsh-llm` 由 DSH 宿主提供。
- **兼容梁神模式**（[dsh-liangshen](https://github.com/linxin666/dsh-liangshen) agent preset）：梁神模式 phase 1 会按白名单剥离 `agent/pre-step` 注入消息，插件检测到梁神模式会话后自动改为把中文指令追加进 persona section——phase 1 的 prompt 过滤器只保留 persona，因此指令在锚定期与晋升后都生效，且不会重复注入。

> 说明：harness 与 DeepSeek API 没有强制推理语言的开关；全中文提示词能最大化概率，但不保证 100%。

## 兼容性

| 项目 | 要求 |
|---|---|
| DSH 会话格式 | **v4**（即 `session.v4.jsonl` 这一代；`v1.1.1` 起不再支持 v4 之前的版本） |
| Node.js | `>=18` |
| 宿主 API | `@deepseek-ai/dsh-llm`（由 DSH 提供，插件不自带） |

### 为什么要求 v4

DSH 的会话格式 v4 要求每条消息的来源由生产方署名：`source.kind` 必须是非空字符串，
且不能是已废弃的 `"plugin"` 包装。因此本插件注入的消息使用：

```js
source: { kind: 'dsh-zh-reasoning', form: 'snapshot', sections: [{ name, text }] }
```

这与官方 `@deepseek-ai/dsh-time-context` 的做法一致。若使用旧写法（`kind: 'plugin'`），
在 v4 会话下写入日志时会失败并提示：

```
format v4 message requires a producer-owned source kind
```

反之，v4 之前的版本使用一份封闭的来源 kind 白名单，不接受自定义 kind，
因此请把本插件用在 v4 及更新的 DSH 上。

## 安装 / 挂载

**方式一：从 GitHub 直接安装（推荐）**

```sh
dsh plugin --profile <name> add "dsh-zh-reasoning@git+https://github.com/ZiYe4u/dsh-zh-reasoning.git"
```

> [!IMPORTANT]
> npm 上的 `dsh-zh-reasoning` 是**上游作者发布的包**（maintainer 为 `graychao`），与本仓库无关。
> 执行 `dsh plugin --profile <name> add dsh-zh-reasoning` 装到的是那个上游版本，而不是本仓库的代码；
> 本仓库也不使用该包名发布到 npm，请务必用上面带 `git+` 的写法。

**方式二：本地开发（`file:` 依赖）** 在 profile 的 `package.json` 中添加依赖并加入 bundles：

```json
{
  "dependencies": { "dsh-zh-reasoning": "file:../dsh-zh-reasoning" },
  "dsh": { "profile": { "bundles": [ "...", "dsh-zh-reasoning" ] } }
}
```

然后在该 profile 目录执行 `pnpm install` 并重启 `dsh web`。

**方式三：直接本地挂载（无需安装）** 把本仓库目录拷贝到 `~/.dsh/profiles/<name>/node_modules/dsh-zh-reasoning`，
再在 `~/.dsh/profiles/<name>/cordis.patch.yml` 中补上挂载行：

```yaml
- insert:
    - id: dsh-zh
      name: dsh-zh-reasoning
```

放在 profile 的 `node_modules` 下时，Node 会沿目录链向上解析到 DSH 自带的
`@deepseek-ai/dsh-llm`，因此不需要额外安装依赖。

## 验证

```sh
dsh --profile <name> --dump-config | findstr dsh-zh
```

看到 `dsh-zh` 行即表示已挂载。之后新建会话，模型应以简体中文思考与回答。

想确认注入真的落盘，可以查看会话日志（`~/.dsh/sessions/<项目>/<session>/session.v4.jsonl.zstd`），
其中应出现 `"source":{"kind":"dsh-zh-reasoning",...}` 的 user 消息。

## 目录结构

```
dsh-zh-reasoning/
├── lib/
│   └── index.js        # 插件入口：pre-step 注入中文 system-reminder（梁神模式下改追加 persona）
├── cordis.patch.yml    # bundle 补丁层：声明插件挂载行
├── package.json        # npm 包清单（含 dsh.bundle.patch 字段，零运行时依赖）
├── CHANGELOG.md        # 更新日志
├── LICENSE             # Apache-2.0 许可证（上游原文，未做修改）
├── .gitattributes      # 统一 LF 换行
├── .gitignore
└── README.md
```

## 开发

本插件不携带依赖，直接挂进 profile 即可调试：

```sh
node -e "import('file:///绝对路径/dsh-zh-reasoning/lib/index.js').then(m=>console.log(m.name))"
```

该命令在 profile 的 `node_modules` 目录树内执行时才能解析到 `@deepseek-ai/dsh-llm`；
若在仓库目录外单独执行，请先自行准备该包，或改用方式一 / 方式二安装。

## 版权与归属

本仓库是**非官方衍生版本（unofficial fork）**，与原作者无隶属或背书关系。
各层级归属如下：

| 层级 | 项目 | 版权 | 许可证 |
|---|---|---|---|
| 上游 | [zhy201810576/dsh-zh-reasoning](https://github.com/zhy201810576/dsh-zh-reasoning) | Copyright 2026 zhy201810576 | Apache-2.0 |
| 更上游 | [imlishiyuan/deepseek-harness-zh-cn](https://github.com/imlishiyuan/deepseek-harness-zh-cn) | Copyright (c) imlishiyuan | Apache-2.0 |
| 本仓库的修改 | [ZiYe4u/dsh-zh-reasoning](https://github.com/ZiYe4u/dsh-zh-reasoning) | Copyright 2026 ZiYe4u | Apache-2.0 |

- `LICENSE` 为上游原文件，**未做任何修改**，其中完整保留了两级上游的版权声明（Apache-2.0 §4(a) / §4(c)）。
- 被修改的源文件（`lib/index.js`）在文件头标注了修改者、修改日期与改动摘要（Apache-2.0 §4(b)）；
  新增与改写的文档（`README.md`、`CHANGELOG.md`、`.gitignore`、`.gitattributes`）在本节统一声明归属。
- 上游与本仓库均**不含 `NOTICE` 文件**，因此不涉及 Apache-2.0 §4(d) 的额外归属展示义务。
- Apache-2.0 §6 不授予商标许可：请勿使用上游作者的名义进行宣传，或暗示其对本项目的背书。
- 本仓库与上游同名，仅表示代码同源，不代表同一项目；如需可辨识度，建议在引用时注明本仓库地址。

## 协议

[Apache-2.0](LICENSE)，保留原作者版权声明。你可以自由使用、修改与再分发，
但需遵守 Apache-2.0 的分发条件（附许可证副本、标注修改、保留原始归属声明）。
