# 多工具 Skill 统一管理方案

## 架构

```
C:\Users\Demon\ai-skills\          ← 统一仓库（Git 管理，推到 GitHub）
  progressive-diagramming/
    SKILL.md
  api-doc-generator/
    SKILL.md
    references/
  grill-me/
    SKILL.md
  ali-java-code-style/
    SKILL.md
```

通过 Windows Junction（目录联接）将统一仓库中的每个 skill 目录链接到各工具的 skill 目录：

| 工具 | Junction 目标路径 |
|------|------------------|
| Codex | `C:\Users\Demon\.codex\skills\<skill-name>` |
| OpenCode | `C:\Users\Demon\.config\opencode\superpowers\skills\<skill-name>` |
| Hermes | `C:\Users\Demon\AppData\Local\hermes\skills\creative\<skill-name>` |

改一处，三端同步。Git 管版本和备份。

## 创建 Junction

必须用 PowerShell，不能用 `cmd //c 'mklink /J ...'`（GBK 编码问题会导致 Python subprocess 解码失败）：

```powershell
New-Item -ItemType Junction -Path "<目标路径>" -Target "<源路径>" -Force
```

## 初始化新 Skill 到三端

1. 在 `ai-skills/` 下创建 skill 目录和 SKILL.md
2. 在三端各创建 Junction 指向统一仓库
3. Git 提交并推送

```bash
cd C:\Users\Demon\ai-skills
git add -A
git commit -m "add: <skill-name>"
git push
```

## 验证 Junction

```bash
# 检查文件可读
for f in /c/Users/Demon/.codex/skills/<name>/SKILL.md \
         /c/Users/Demon/.config/opencode/superpowers/skills/<name>/SKILL.md \
         /c/Users/Demon/AppData/Local/hermes/skills/creative/<name>/SKILL.md; do
  [ -f "$f" ] && echo "OK: $f" || echo "MISSING: $f"
done

# 检查写同步：在统一仓库写一行，三端应都能读到
echo "<!-- test -->" >> /c/Users/Demon/ai-skills/<name>/SKILL.md
tail -1 /c/Users/Demon/.codex/skills/<name>/SKILL.md
tail -1 /c/Users/Demon/.config/opencode/superpowers/skills/<name>/SKILL.md
tail -1 /c/Users/Demon/AppData/Local/hermes/skills/creative/<name>/SKILL.md
# 清理测试标记
sed -i '/<!-- test -->/d' /c/Users/Demon/ai-skills/<name>/SKILL.md
```

## 与 Nacos AI Registry 的区别

| 维度 | 本地 Junction + Git | Nacos AI Registry |
|------|---------------------|-------------------|
| 适用场景 | 个人多工具 | 团队多环境 |
| 部署成本 | 零 | 需部署或用云服务 |
| 版本管理 | Git | 内置版本/标签/审核 |
| 权限隔离 | 无 | 命名空间+可见性 |
| 安全扫描 | 无 | 内置 |
| 推荐 | 个人 vibe coding | 团队生产环境 |

## 注意事项

1. Junction 只同步 `ai-skills/` 下的内容，外部脚本（如 `~/.wiki-17u/scripts/`）不在同步范围
2. Git 会提示 LF/CRLF 转换警告，这是正常的，`.gitattributes` 会处理
3. 新建 skill 时记得在三端都创建 Junction，否则只有统一仓库有文件
4. GitHub 仓库地址：`git@github.com:demonXh/ai-skills.git`
