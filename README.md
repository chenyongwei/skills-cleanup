# Cleanup Skills

一组可以独立使用的代码仓库清理 Skill。

| Skill | 用途 |
| --- | --- |
| [cleanup](skills/cleanup/SKILL.md) | 清理死代码、临时垃圾文件，以及失实、过时或冗余的注释和文档。默认先出报告，确认后再删，并验证行为未变。 |

## 目录约定

每个 Skill 独立放在 `skills/<skill-name>/`，入口为 `SKILL.md`。只有需要时，才在该 Skill 目录内添加 `scripts/`、`references/` 或 `assets/`。根目录的 `.claude-plugin/plugin.json` 是整个集合的插件清单，不属于某个单独的 Skill。
