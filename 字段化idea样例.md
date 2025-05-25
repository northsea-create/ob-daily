- idea_content:: 我想做一个 AI 辅助的零碎知识点收集器，用于快速捕捉和分类我在阅读、对话、听播客中的灵光一现，目前我用 flomo，但总觉得缺“结构”和“复用感”。
- idea_trigger:: 每天都有大量知识碎片和句子浮现，却没有一个系统能持续性地整理它们、并连接到已有知识体系里。
- idea_validate:: 试着用一个 Obsidian 插件 + GPT，把 flomo 的导出内容结构化进我的知识库，观察复用和聚合效果。
- idea_status:: 暂存
- idea_project:: [[idea-知识管理]]



---

```dataview
table idea_content as "内容", idea_trigger as "触发", idea_validate as "验证", idea_status as "状态", idea_project as "关联项目", file.name as "来源"
from ""
where idea_content
sort file.name desc
```