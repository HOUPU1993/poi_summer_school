# Benchmarking Multi-Source POI Data — Site Source

这是一个 **Jekyll** 项目，GitHub Pages 原生支持，push 上去就会自动构建部署，不需要任何额外配置或构建工具。
内容全部是 Markdown 文件，你可以直接在 GitHub 网页上点开文件、点铅笔图标编辑，改完提交（commit）网站就会自动重新构建。

---

## 目录结构

```
.
├── _config.yml           # 站点配置（标题、markdown 引擎等）
├── _layouts/
│   └── default.html      # 页面外壳：侧边栏导航 + 页头 + 页脚（一般不需要改）
├── assets/
│   ├── css/style.css     # 所有样式
│   ├── poi_study/        # POI Study 论文里的图（已从 PDF 提取好）
│   └── poibench/         # POIBench 论文里的图
├── index.md              # 首页（Hero + 三个 Part 的入口卡片）
├── research-journey.md   # Part I 内容
├── poi-study.md          # Part II 内容
├── poibench.md           # Part III 内容
├── Gemfile               # 可选，本地预览用
└── .gitignore
```

**每个 `.md` 文件就是一个页面**，页面顶部有一段 `---` 包裹的 front matter，指定标题和网址（permalink），下面就是正文。正文基本都是标准 Markdown 语法（标题、段落、列表、表格、图片、链接），只有少数几处用了自定义样式（见下面"常用编辑模式"）。

---

## 常用编辑模式速查

网站里几种好看的卡片/提示框/步骤条，都不需要写 HTML，用 Markdown + 一个 `{: .class}` 标记就行：

### 普通段落、标题、表格、图片
直接用标准 Markdown 语法，改这些内容最简单，随便改。

### 高亮提示框（callout）
```markdown
> **提示标题**
>
> 正文内容...
{: .callout}
```

### 卡片网格（数字/统计卡片）
```markdown
- **①** — 说明文字
- **②** — 说明文字
{: .card-grid}
```

### 编号步骤条
```markdown
1. **步骤标题**<br>步骤说明文字
2. **步骤标题**<br>步骤说明文字
{: .steps}
```

### 图片 + 图注
```markdown
![替代文字](../assets/poi_study/xxx.png)
*<span class="fig-label">FIG. 1</span>图注文字*
{: .figcap}
```
> 注意：`poi-study.md` / `poibench.md` / `research-journey.md` 这三个页面里，图片路径都用 `../assets/...`（往上一层再进 assets），因为这些页面本身"住"在 `/poi-study/` 这样的子目录里。如果你新增图片页面，记得也用 `../assets/...` 这种相对路径，不要写成 `/assets/...`，否则以后仓库改名或者部署路径变化时图片会失效。

### 数据集小标签（badge）
```markdown
*SafeGraph*{: .badge-safegraph}
```
可用的类名：`.badge-overture` `.badge-safegraph` `.badge-foursquare` `.badge-osm` `.badge-gplaces`

### 两张图并排
```markdown
<div class="fig-grid" markdown="1">

<div markdown="1">
![图A](../assets/xxx.png)
*<span class="fig-label">FIG. 2a</span>说明*
{: .figcap}
</div>

<div markdown="1">
![图B](../assets/xxx.png)
*<span class="fig-label">FIG. 2b</span>说明*
{: .figcap}
</div>

</div>
```
这是唯一一处需要写一点 HTML 的地方（Markdown 本身没有"并排两栏"的语法）。照抄这个模板换图片和文字就行，注意 `<div markdown="1">` 后面一定要空一行，`</div>` 前面也要空一行，不然里面的 Markdown 不会被正确解析。

### 侧边栏导航
在 `_layouts/default.html` 里手动写死的（对应各页面标题里的 `{#custom-id}`）。如果你新增了一个 `## 标题 {#my-id}`，想让它出现在左侧导航，去 `_layouts/default.html` 里对应 Part 下面加一行：
```html
<a href="{{ '/poi-study/#my-id' | relative_url }}">我的小节标题</a>
```

---

## 本地预览（可选）

如果你想在推到 GitHub 之前先在自己电脑上看效果，装好 Ruby 后执行：

```bash
gem install jekyll
jekyll serve
```
然后打开 `http://localhost:4000` 就能看到和线上一样的效果。每次改完 `.md` 文件保存，页面会自动重新生成（刷新浏览器即可）。

如果不想装 Ruby 环境，也可以跳过本地预览，直接改完推到 GitHub，等它自动构建（通常 1 分钟内），然后在 Pages 给的网址上看效果。

---

## 部署到 GitHub Pages

1. 把这个文件夹整个 push 到你的 GitHub 仓库（作为仓库根目录内容）。
2. 进仓库 **Settings → Pages**，Source 选择 `Deploy from a branch`，Branch 选 `main`（或你用的分支）+ `/ (root)`，保存。
3. 等 1-2 分钟，Pages 会给你一个类似 `https://你的用户名.github.io/仓库名/` 的网址。

> 不需要在 `_config.yml` 里手动设置 `baseurl` —— 侧边栏导航和样式表已经用 Jekyll 的 `relative_url` 处理过，会自动适配 `仓库名` 这段路径；正文里的图片和跳转链接用的是相对路径，同样不受影响。

### 上课演示 → 一周后转为 private
GitHub Pages 只有仓库是 **public** 时才能免费用（除非你是 GitHub Pro/Team/Enterprise 账号）。也就是说，把仓库设为 private 之后，这个网站会自动下线（访问会 404），但代码和内容都还在仓库里，不会丢，你随时可以再切回 public 让它重新上线。

---

## 内容来源

- **Part I** 内容整理自你的研究日志 `1_The_Comparison_of_POI_Data_Resources.md`（Week 1–6）
- **Part II** 内容整理自 *A Multi-Dimensional Quality Assessment of Popular Alternative POI Datasets*（POI Study 论文），图表直接从论文 PDF 中提取
- **Part III** 内容整理自 *POIBench: An Interactive Benchmarking Platform for Multi-Source POI Dataset Evaluation*（SIGSPATIAL '26 Demo），图表同样从论文 PDF 中提取

如果论文有更新版本，只需要替换 `assets/poi_study/` 或 `assets/poibench/` 里对应的图片文件（文件名保持一致即可），或者直接编辑 `poi-study.md` / `poibench.md` 里的文字和数字。
