# Wiki 写入集成踩坑记录

## 背景

曾尝试将 api-doc-generator 生成的接口文档直接写入 17u Wiki（wiki.17u.cn / toca.17u.cn）。

## 失败原因

脚本 `setup.mjs` 使用 Node.js `readline` 模块等待 stdin 输入 Cookie：

```js
const rl = createInterface({ input: process.stdin, output: process.stdout });
rl.question('请粘贴 Cookie 字符串: ', (cookie) => { ... });
```

Codex Agent 运行脚本时无法处理交互输入，`readline.question()` 永远不会返回，导致 Codex 默认 600s 超时后被 Stop hook 终止。

## 修复方案

1. 脚本支持 `--cookie` CLI 参数，Agent 非交互式传入
2. 支持 `--non-interactive` 模式，无参数时打印提示并退出码 1
3. 认证信息存入 `~/.wiki-17u/config.json`

```bash
# Agent 友好方式
node setup.mjs --cookie "JSESSIONID=xxx; uid=yyy"

# 非交互模式（打印提示后退出）
node setup.mjs --non-interactive
```

## 通用规则

任何被 Agent 调用的脚本都不可使用：
- `readline`
- `prompt`
- `process.stdin.on('data')`
- 任何等待 stdin 的方式

替代方案：用 CLI 参数、环境变量或配置文件传值。

## 当前状态

Wiki 写入功能已从 SKILL.md 中移除，`~/.wiki-17u/` 目录已删除。如需重新实现，按上述修复方案重建脚本即可。
