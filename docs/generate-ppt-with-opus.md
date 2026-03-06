# 用 Cursor + Opus 生成专业 PPT

## 背景

手动做 PPT 费时费力，特别是技术方案类的演示文稿。通过 Cursor + Claude Opus，可以从一份粗略的文字大纲直接生成结构完整、风格统一的 `.pptx` 文件。

## 前置准备

### 环境

- **Cursor** 编辑器（配置 Claude Opus 模型）
- **Python 3.x**
- **python-pptx** 库

### 安装依赖

```bash
pip install python-pptx
```

## 操作流程

### 第一步：准备内容大纲

不需要写得很正式，把核心要点列出来就行。例如：

```
目的：在外部搭建使用AI工具的pipeline

前期：
- 工具开发：每日arxiv论文自动阅读，Paper List维护
- 代码开发：自动驾驶白盒逻辑验证，仿真器+Agent对抗

中期：
- 具身智能课题，团队协作模式探索
- 自动驾驶单点技术验证
- GPU转昇腾代码迁移Agent

后期：
- AI自闭环完成需求、实验、论文

资源申请：AI工具会员、GPU/NPU算力、人力
```

### 第二步：让 Opus 生成 PPT

在 Cursor 中直接对话，把大纲发给 AI，并说明需求：

> "帮我用 python-pptx 生成一份 PPT，内容是这些，深色科技风格，呈现给领导的"

Opus 会：
1. 自动编写 Python 脚本（使用 `python-pptx` 库）
2. 设计配色方案、页面布局、卡片样式
3. 将你的内容结构化地填入各个 slide
4. 直接运行脚本生成 `.pptx` 文件

### 第三步：检查和微调

生成的 PPT 可以直接用 PowerPoint 或 WPS 打开。通常需要微调的地方：
- 替换为公司 PPT 模板
- 调整个别文字措辞
- 添加公司 Logo

## 关键代码片段

### 基本结构

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor

prs = Presentation()
prs.slide_width = Inches(13.333)  # 16:9 宽屏
prs.slide_height = Inches(7.5)

# 使用空白布局，完全自定义
slide = prs.slides.add_slide(prs.slide_layouts[6])
```

### 设置深色背景

```python
def set_slide_bg(slide, color):
    bg = slide.background
    fill = bg.fill
    fill.solid()
    fill.fore_color.rgb = color

set_slide_bg(slide, RGBColor(0x1B, 0x1F, 0x3B))
```

### 添加圆角卡片

```python
from pptx.enum.shapes import MSO_SHAPE

shape = slide.shapes.add_shape(
    MSO_SHAPE.ROUNDED_RECTANGLE,
    left, top, width, height
)
shape.fill.solid()
shape.fill.fore_color.rgb = RGBColor(0x24, 0x29, 0x4E)
shape.line.fill.background()  # 无边框
shape.adjustments[0] = 0.02   # 圆角半径
```

### 添加文字

```python
from pptx.enum.text import PP_ALIGN

tb = slide.shapes.add_textbox(left, top, width, height)
tf = tb.text_frame
tf.word_wrap = True
p = tf.paragraphs[0]
p.text = "标题文字"
p.font.size = Pt(32)
p.font.color.rgb = RGBColor(0xFF, 0xFF, 0xFF)
p.font.bold = True
p.font.name = "Microsoft YaHei"
p.alignment = PP_ALIGN.LEFT
```

## 效果

最终生成了一份 8 页的深色科技风 PPT，包含：

| 页码 | 内容 |
|------|------|
| 1 | 封面 |
| 2 | 背景与动机 |
| 3 | 总体规划路线（三阶段卡片） |
| 4 | 前期：基础验证（工具+代码双卡片） |
| 5 | 中期：团队协作与技术突破 |
| 6 | 后期：AI 自闭环研发（流程图） |
| 7 | 资源需求 |
| 8 | 总结与下一步 |

生成的文件：[AI_Pipeline_Proposal.pptx](../files/AI_Pipeline_Proposal.pptx)

## 小结

- **耗时**：从大纲到成品 PPT，约 2 分钟
- **质量**：结构清晰、配色统一，可直接用于汇报（微调后更佳）
- **适用场景**：技术方案、项目汇报、工作总结等模式化 PPT
- **局限**：复杂动画和精细排版仍需手动调整
