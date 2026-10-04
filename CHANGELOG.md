# 更新日志

本项目遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## 1.1.1

### 修复

- **修复 v4 会话下“本轮运行失败”**：注入消息的来源不再使用 v4 已废弃的
  `source.kind: "plugin"`，改为生产方署名 `source.kind: "dsh-zh-reasoning"`，
  避免持久化时抛出 `format v4 message requires a producer-owned source kind`。
- `agent/pre-step` 处理器改为展开上游 `decision` 对象（`{ ...decision, messages: [...] }`），
  不再重建 `{ kind: 'enter', messages }`，以免丢失 `startsRequestSeries` 等下游字段。

### 变更

- 运行时依赖改为纯 peer：移除 `dependencies` 中的 `@deepseek-ai/dsh-llm`，
  由 DSH 宿主提供；仓库不再需要 `node_modules`，可直接克隆使用。
- `package.json` 补充 `exports` / `engines` / `repository` / `homepage` / `bugs` 字段。
- 包版本号由 `0.1.1` 调整为 `1.1.1`，与 npm 上 `1.1.0` 的版本线对齐。
- 仓库更名为 `ZiYe4u/dsh-zh-reasoning-fixed`，插件包名与注册名同步改为
  `dsh-zh-reasoning-fixed`（`source.kind` 随之变化），以避免与上游
  `zhy201810576/dsh-zh-reasoning` 及 npm 上的同名包混淆。

### 文档

- README 新增「兼容性」章节，说明 v4 的 source 署名要求与 v4 之前的版本限制。
- 补充梁神模式兼容说明、零依赖安装方式与注入结果的自查方法。

## 1.1.0 及更早版本

早期版本使用 `source.kind: "plugin"` 的注入写法，仅适用于 v4 之前的会话格式；
更早的版本记录未在本仓库维护。
