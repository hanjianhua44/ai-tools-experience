# 用 Cursor + Opus 阅读论文并生成讲解 PPT

## 背景

阅读学术论文后做讲解 PPT 是科研和技术分享中的常见需求。传统流程需要先通读论文、提炼要点、再手动排版 PPT，耗时且重复。通过 Cursor + Claude Opus，可以一步到位：AI 直接阅读 PDF 论文，提取关键内容，并自动生成带原文图表的讲解 PPT。

## 前置准备

### 环境

- **Cursor** 编辑器（配置 Claude Opus 模型）
- **Python 3.x**
- **python-pptx** — 生成 PPT
- **PyMuPDF (fitz)** — 读取 PDF 并提取/裁剪图片

### 安装依赖

```bash
pip install python-pptx PyMuPDF
```

## 操作流程

### 第一步：把论文 PDF 放到工作目录

将 PDF 文件放到 Cursor 能访问的路径下，例如：

```
D:\projects\test\paper.pdf
```

### 第二步：让 Opus 阅读论文

在 Cursor 中直接对话：

> "D:\projects\test\paper.pdf，这个论文你可以阅读吗，可以帮我做成一个讲解 paper 的 PPT 吗"

Opus 会：
1. 使用内置 PDF 读取工具，将论文内容转为文本
2. 理解论文结构（摘要、方法、实验、结论等）
3. 自动编写 Python 脚本生成 PPT

### 第三步：要求加入论文原图

生成初版后，可以追加要求：

> "论文里的图是可以加进去的吗，让 PPT 看的更形象点"

Opus 会：
1. 使用 **PyMuPDF** 从 PDF 中按页面区域裁剪出各个 Figure
2. 将裁剪出的高清图片嵌入到对应的 PPT 页面中
3. 调整布局以图文并茂地呈现

### 第四步：检查和微调

生成的 PPT 可以直接用 PowerPoint 打开。通常可微调：
- 个别文字措辞
- 图片位置和大小
- 添加自己的总结/评论页

## 关键代码片段

### 从 PDF 裁剪指定区域的图表

```python
import fitz

doc = fitz.open("paper.pdf")
page = doc[0]  # 第 1 页

# 指定裁剪区域 (left, top, right, bottom)，单位为 PDF points
clip = fitz.Rect(0, 50, 612, 390)

# 高分辨率渲染
mat = fitz.Matrix(3.0, 3.0)
pix = page.get_pixmap(matrix=mat, clip=clip)
pix.save("fig1_overview.png")
```

### 将图片嵌入 PPT

```python
from pptx import Presentation
from pptx.util import Inches

prs = Presentation()
prs.slide_width = Inches(13.333)
prs.slide_height = Inches(7.5)

slide = prs.slides.add_slide(prs.slide_layouts[6])
slide.shapes.add_picture(
    "fig1_overview.png",
    left=Inches(0.5),
    top=Inches(1.9),
    width=Inches(12.3)
)
```

### 深色主题 + 图文混排的 Slide 结构

```python
from pptx.dml.color import RGBColor
from pptx.enum.shapes import MSO_SHAPE

# 深色背景
bg = slide.background.fill
bg.solid()
bg.fore_color.rgb = RGBColor(0x1B, 0x1B, 0x2F)

# 左侧放文字分析
add_rect(slide, Inches(0.7), Inches(1.8), Inches(5.5), Inches(5.0),
         RGBColor(0x22, 0x22, 0x3A))
# 文字内容...

# 右侧放论文原图
slide.shapes.add_picture("fig7_performance.png",
    Inches(6.5), Inches(1.7), width=Inches(6.3))
```

## 效果

最终生成 16 页深色科技风讲解 PPT，包含论文原图：

| 页码 | 内容 | 嵌入图 |
|------|------|--------|
| 1 | 封面（标题、作者、机构） | — |
| 2 | Outline 大纲 | — |
| 3 | Background & Motivation | — |
| 4 | System Overview | Fig. 1 工作流对比 |
| 5 | Core Contributions | — |
| 6 | Hardware Architecture | Fig. 2 系统设计 |
| 7 | Software Architecture | — |
| 8 | Robot-Free Policy Iteration | Fig. 3 流程架构 |
| 9 | AR Visual Foresight & Pipeline | — |
| 10 | System Verification | Fig. 6 Scaling Laws |
| 11 | Evaluation Tasks | Fig. 4 任务展示 |
| 12 | Performance Comparison | Fig. 7 性能对比 |
| 13 | Distributed Generalization | Fig. 8 分布式结果 |
| 14 | User Study | Fig. 9/10 用户研究 |
| 15 | Conclusion & Limitations | — |
| 16 | Thank You | — |

生成的文件：[RoboPocket_Paper_Presentation.pptx](../files/RoboPocket_Paper_Presentation.pptx)

## 小结

- **耗时**：从 PDF 到完整带图 PPT，约 5 分钟（含图片提取）
- **质量**：自动提炼论文结构，图文并茂，可直接用于组会/论文分享
- **适用场景**：论文讲解、文献综述、技术分享等
- **核心能力**：
  - Opus 可直接读取 PDF 全文内容
  - PyMuPDF 按区域裁剪论文图表，保持高清
  - python-pptx 生成带图片的精美 PPT
- **局限**：纯扫描件 PDF 可能无法提取文本；非常复杂的排版需手动微调
