![Modern Robotics — Mechanics, Planning, and Control 学习笔记](assets/cover.svg)

# Modern Robotics 学习笔记

<p align="center">
  <a href="#english">English</a> |
  <strong>中文</strong>
</p>

<a id="chinese"></a>

围绕 Kevin M. Lynch 与 Frank C. Park 的 *Modern Robotics: Mechanics, Planning, and Control* 整理的中文学习笔记。按教材章节保存概念解释、公式推导、配图与 Q&A，让阅读、练习和复习有迹可循。

**[章节索引](#章节索引)** · **[教材 PDF](books/modern-robotics.pdf)** · **[学习进度](progress.md)** · **[来源与页码](sources.md)**

## 章节索引

每章保留一份正文笔记和一份配套 Q&A。正文适合连贯阅读；Q&A 用于自测、查看参考答案与回顾易错点。

| 章节 | 主要内容 | 阅读入口 | 配套练习 |
| :--- | :--- | :--- | :--- |
| **02 · 构型空间**<br>Configuration Space | 自由度、关节约束、拓扑、任务空间与工作空间 | [章节笔记](notes/ch02/ch02-configuration-space.md) | [Q&A](notes/ch02/ch02-configuration-space-qa.md) |
| **03 · 刚体运动**<br>Rigid-Body Motions | 旋转、齐次变换、螺旋理论、指数与对数、力旋量 | [章节笔记](notes/ch03/ch03-rigid-body-motions.md) | [Q&A](notes/ch03/ch03-rigid-body-motions-qa.md) |
| **04 · 正向运动学**<br>Forward Kinematics | 空间与物体形式 PoE、机械臂建模、URDF | [章节笔记](notes/ch04/ch04-forward-kinematics.md) | [Q&A](notes/ch04/ch04-forward-kinematics-qa.md) |

> [!NOTE]
> 本表列出已经建立笔记的章节。当前小节、待答练习与下一步统一见 [学习进度](progress.md)；笔记覆盖与独立掌握分别记录。

<details>
<summary><strong>展开全书目录</strong></summary>

以下目录用于定位教材内容；未列出阅读链接的章节尚未建立笔记。

| 章节 | 主题 |
| :--- | :--- |
| 01 | 概览 · Preview |
| 02 | [构型空间 · Configuration Space](notes/ch02/ch02-configuration-space.md) |
| 03 | [刚体运动 · Rigid-Body Motions](notes/ch03/ch03-rigid-body-motions.md) |
| 04 | [正向运动学 · Forward Kinematics](notes/ch04/ch04-forward-kinematics.md) |
| 05 | 速度运动学与静力学 · Velocity Kinematics and Statics |
| 06 | 逆向运动学 · Inverse Kinematics |
| 07 | 闭链运动学 · Kinematics of Closed Chains |
| 08 | 开链动力学 · Dynamics of Open Chains |
| 09 | 轨迹生成 · Trajectory Generation |
| 10 | 运动规划 · Motion Planning |
| 11 | 机器人控制 · Robot Control |
| 12 | 抓取与操作 · Grasping and Manipulation |
| 13 | 轮式移动机器人 · Wheeled Mobile Robots |
| 附录 A–D | 常用公式、其他旋转表示、D–H 参数、优化与拉格朗日乘子 |

</details>

## 如何阅读

1. **先读正文。** 从章首目录进入对应小节，沿“问题 → 概念与推导 → 例子”阅读；教材图在使用处直接展示。
2. **再做自测。** 打开同章 Q&A，先尝试回答题目，再对照参考答案。看过答案与能独立完成分别判断。
3. **需要时回到原书。** 图号、公式编号和书页／PDF 页码帮助定位原文；版本与页码规则见 [来源索引](sources.md)。

笔记采用 Markdown 与 LaTeX，使用相对链接连接正文、Q&A 和插图，可在 GitHub 浏览，也可在 Obsidian 中打开同一目录。每章正文和 Q&A 均提供折叠目录与互相跳转的入口。

## 教材与配套资源

| 资源 | 用途 |
| :--- | :--- |
| [仓库内教材 PDF](books/modern-robotics.pdf) | 当前笔记依据的 2017 年 5 月 3 日预印本，共 644 个 PDF 页面 |
| [作者官方书籍主页](https://hades.mech.northwestern.edu/index.php/Modern_Robotics) | 获取作者发布的预印本、勘误与配套材料 |
| [官方视频讲解](https://modernrobotics.northwestern.edu/nu-gm-book-resource/) | 按章节回看概念和示例 |

笔记页码以仓库中的 PDF 为准；更换教材版本时，请重新核对页码。教材与引用图片的权利归原作者及相应权利人所有；本仓库为个人学习记录。

<details>
<summary><strong>仓库结构与维护方式</strong></summary>

```text
modern-robotics/
├── README.md                  # 阅读首页与章节索引
├── README_EN.md               # English entry page
├── books/
│   └── modern-robotics.pdf     # 当前学习版本的教材
├── notes/
│   └── chXX/
│       ├── chXX-<topic>.md     # 章节正文
│       └── chXX-<topic>-qa.md  # 配套问题与参考答案
├── attachments/chXX/          # 按章存放的教材图
├── assets/                    # 首页视觉素材
├── sources.md                 # 教材版本、页码与来源覆盖
├── progress.md                # 唯一实时学习断点
├── learning_protocol.md       # 教学节奏与笔记规范
└── AGENTS.md                  # 学习助手的恢复入口
```

新增章节时沿用配对文件命名，将教材图放入同章附件目录，并在首页补上阅读入口。长期笔记写入正文，问题与参考答案写入 Q&A，实时状态只维护在 `progress.md`。

续学时，在本仓库目录中发送：

> 继续学习 Modern Robotics。请先读取仓库根目录的 AGENTS.md，按其中指引从当前断点继续。

教学节奏、来源核对与状态维护见 [学习协议](learning_protocol.md)。项目层面的路线规划与章节取舍由独立的路线规划对话维护。

</details>

---

<a id="english"></a>

## English

<p align="center">
  <strong>English</strong> |
  <a href="#chinese">中文</a>
</p>

Chinese study notes for Kevin M. Lynch and Frank C. Park's *Modern Robotics: Mechanics, Planning, and Control*. The repository follows the textbook chapter by chapter, combining concept explanations, derivations, figures, and self-check questions in one place.

The detailed notes are written primarily in Chinese, with English chapter names and key terms included for cross-language reference. Each completed chapter has a paired note and Q&A file so that reading and retrieval practice stay connected.

**[Chapter Index](#english-chapter-index)** · **[Textbook PDF](books/modern-robotics.pdf)** · **[Progress](progress.md)** · **[Sources](sources.md)**

<a id="english-chapter-index"></a>

### Chapter Index

Each chapter keeps a main note and a companion Q&A. Read the note for a continuous explanation, then use the Q&A for retrieval practice and review.

| Chapter | Topics | Notes | Practice |
| :--- | :--- | :--- | :--- |
| **02 · Configuration Space** | Degrees of freedom, joint constraints, topology, task and workspace | [Chapter note](notes/ch02/ch02-configuration-space.md) | [Q&A](notes/ch02/ch02-configuration-space-qa.md) |
| **03 · Rigid-Body Motions** | Rotations, homogeneous transformations, screw theory, exponentials and logarithms, wrenches | [Chapter note](notes/ch03/ch03-rigid-body-motions.md) | [Q&A](notes/ch03/ch03-rigid-body-motions-qa.md) |
| **04 · Forward Kinematics** | Space and body PoE, robot modeling, and URDF | [Chapter note](notes/ch04/ch04-forward-kinematics.md) | [Q&A](notes/ch04/ch04-forward-kinematics-qa.md) |

> [!NOTE]
> This table lists chapters with notes already available. The current section, pending exercises, and next action are tracked in [Progress](progress.md). Coverage and independent mastery are recorded separately.

## How to Use This Repository

1. **Read the chapter note first.** Follow the section links from the chapter index and move from problems to concepts, derivations, and examples.
2. **Practice with the paired Q&A.** Answer each question before opening the reference answer, and distinguish recognition from independent recall.
3. **Return to the textbook when needed.** Figure numbers, equation numbers, and page references point back to the local PDF; see the [source index](sources.md) for version details.

The notes use Markdown and LaTeX with relative links between chapters, Q&A files, and figures. They can be read directly on GitHub or opened as an Obsidian vault.

## Textbook and Resources

| Resource | Use |
| :--- | :--- |
| [Textbook PDF in this repository](books/modern-robotics.pdf) | The May 3, 2017 preprint used for these notes; 644 PDF pages |
| [Official book page](https://hades.mech.northwestern.edu/index.php/Modern_Robotics) | Preprint, errata, and companion materials from the authors |
| [Official video lectures](https://modernrobotics.northwestern.edu/nu-gm-book-resource/) | Chapter-by-chapter explanations and examples |

Page references use the PDF stored in this repository. Recheck page numbers when changing textbook versions. Copyright for the textbook and reproduced figures remains with the authors and respective rights holders; this repository is a personal study record.

<p align="right"><a href="#chinese">Back to 中文</a></p>
