-idea_content:: 我想打造一个 AI 创作者小屋，它像“记忆宫殿”一样，把提示词、工具、课文等抽象内容具象化成角色、物件、场景，让人可以在其中“游逛”并触发内容。
-idea_trigger:: 灵感来自我自己在构建提示词系统时的混乱，觉得视觉化和空间感能帮助记忆，也能增强表达感。
-idea_validate:: 快速做一个 HTML 小屋原型，加入 3 个提示词角色和一个教学模块，看看视觉效果如何，是否具有表达力。
idea_status:: 暂存
-idea_project:: [[idea-创作宫殿]]

-idea_content:: 我想做一个 HTML 生日贺卡生成器，允许用户选择主题、角色、配乐和祝福语，生成一份个性化的祝福网页，适合朋友间表达情感。
-idea_trigger:: 在准备朋友生日祝福时总觉得微信文本或图片太普通，想要更温柔、有仪式感又能复用的形式。
-idea_validate:: 用静态 HTML/CSS 写一个最小可用版，预设两个主题和祝福句子，分享给一个朋友试试反馈。
-idea_status:: 暂存
-idea_project:: [[idea-创作礼物]]

-idea_content:: 我想构建一个 AI 目标规划助手，它能根据我的目标设定、历史日记和完成情况，自动拆解出第二天的优先任务和计划，形成个人节奏感。
-idea_trigger:: 常常写了很多计划却无法高效执行，或者忘了长期目标，现在需要一个像“战术助手”一样的存在帮我组织节奏。
-idea_validate:: 整理我现有的一周任务 + 日记结构，尝试用 prompt + GPT4 来推导出第二天的安排，看看是否有效。
-idea_status:: 验证中
-idea_project:: [[idea-ai贾维斯]]

-idea_content:: 我想做一个 API 创意爆发工具，只要用户说“我有 XX + XX 两个 API”，就能自动提示可能的组合产品创意，并支持反向操作：想做 XX 类产品，该调哪些 API。
-idea_trigger:: 来自小排老师的分享——调 2 个 API 就有 10000 种组合，而我总感觉自己缺创意，其实是少了组合机制和提示系统。
-idea_validate:: 整理 10 个 API 的功能简介，喂给 GPT 让它生成组合产品创意，观察输出质量和提示词优化空间。
-idea_status:: 暂存
-idea_project:: [[idea-api灵感脑暴器]]

-idea_content:: 我想做一个 AI 辅助的零碎知识点收集器，用于快速捕捉和分类我在阅读、对话、听播客中的灵光一现，目前我用 flomo，但总觉得缺“结构”和“复用感”。
-idea_trigger:: 每天都有大量知识碎片和句子浮现，却没有一个系统能持续性地整理它们、并连接到已有知识体系里。
-idea_validate:: 试着用一个 Obsidian 插件 + GPT，把 flomo 的导出内容结构化进我的知识库，观察复用和聚合效果。
-idea_status:: 暂存
-idea_project:: [[idea-知识管理]]



---

```dataview
table idea_content as "内容", idea_trigger as "触发", idea_validate as "验证", idea_status as "状态", idea_project as "关联项目", file.name as "来源"
from ""
where idea_content
sort file.name desc
```