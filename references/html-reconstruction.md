# 设计稿还原为 HTML / 小程序的素材策略

当用户想把生成图、截图或设计稿还原成 HTML/CSS，或直接还原成微信小程序 WXML/WXSS 时，先判断元素应该**代码化**还是**素材化**。先沿用设计阶段的组件、字体、间距、素材边界与适配说明；目标是让内容可编辑、交互可用、布局能适配，再匹配视觉。已有参考稿要求原样还原时尊重其风格，不默认增加或移除装饰。

## 元素分类

| 元素 | 推荐方式 | 原因 |
|------|---------|------|
| 页面布局、卡片、表格、按钮、输入框、筛选器、文字 | HTML/CSS | 几何结构清晰，代码更稳定 |
| 常规线性图标（日历、筛选、刷新、设置、导航图标） | SVG / icon library / CSS | 线宽、颜色、尺寸可控，图生图容易变形 |
| Logo、摄影、原创插画、确有必要的复杂品牌装饰 | 优先原始高质量素材，缺失时再生成或重绘 | 保持品牌准确与清晰边界，避免重复生产素材 |
| 数据图表、地图、关系图、需要交互的产品面板 | 组件、图表库、SVG 或画布绘制 | 数值、标签和状态必须准确且可变化；不能仅因复杂就贴图 |
| 已确认设计中的背景与装饰 | 简单效果用CSS，复杂效果复用独立素材 | 默认不新增纹理，装饰不能承载正文或控件 |

## 微信小程序适配

微信小程序高保真还原时，不要先生成 HTML 再机械转换成 WXML。更稳的方式是直接用小程序的文件模型落地：

- `*.wxml`：页面结构、卡片、按钮、列表、文案层级。
- `*.wxss`：rpx 尺寸、字体、颜色、阴影、圆角、定位和响应式。
- `*.ts` / `*.js`：状态、交互和页面数据。
- `${miniprogramRoot}/assets/`：复杂插画、logo、空状态、hero 装饰、纹理和图生图重绘后的素材。若配置中的 `miniprogramRoot` 是 `miniprogram/`，实际目录就是 `miniprogram/assets/`，WXML 的资源路径从该根目录写起。

小程序里推荐这样分工：

| 元素 | 小程序实现方式 |
|------|---------------|
| 页面布局、卡片、按钮、tab、文本 | WXML/WXSS，优先使用 rpx 和现有组件库 |
| 简单线性图标 | 组件库图标、SVG、iconfont 或小 PNG |
| 插画、复杂渐变、玻璃拟态、3D/拟物元素 | 独立 PNG/WebP/SVG 资产，用 `<image>` 放置 |
| 背景纹理和柔光 | 优先图片素材；简单效果可用多层 `linear-gradient` / `radial-gradient` 近似 |
| 需要等比缩放的主视觉 | `<image mode="aspectFit">` 或 `mode="widthFix"`，固定容器尺寸 |

小程序素材建议目录：

```text
miniprogram/assets/ui/
miniprogram/assets/illustrations/
miniprogram/assets/icons/
miniprogram/assets/generated/
```

如果参考图里有复杂插画，不要用 WXSS 硬画，也不要直接裁低清图塞进页面。走这条链路：

```text
局部裁图参考 -> 图生图重绘高清素材 -> 抠图/清边/裁边 -> 放入 assets -> WXML image 引用 -> 开发者工具截图验证
```

小程序里引用素材示例（`mode` 每次只能取一个合法值）：

```xml
<image class="hero-illustration" src="/assets/illustrations/ai-tools-hero.png" mode="aspectFit" />
```

```xml
<image class="banner" src="/assets/illustrations/banner.png" mode="widthFix" />
```

```css
.hero-illustration {
  width: 320rpx;
  height: 240rpx;
  position: absolute;
  right: 32rpx;
  top: 120rpx;
}
```

小程序验证不要用普通浏览器截图替代。优先使用微信开发者工具模拟器，并固定同一设备预设、页面 viewport 宽高和 DPR；截图对比只取页面可视区，排除系统状态栏、胶囊按钮和开发者工具栏。自动化时可以用 `miniprogram-automator` 连接开发者工具，打开指定页面、截图，再用 pixel diff 和人工 side-by-side 检查。浏览器验证只适合 HTML 原型，不代表小程序真实渲染。

## 关键判断

优先复用已有组件、图标库、字体与原始素材。布局、动态数据、状态和操作必须保持可编辑；照片、插画与特殊装饰放在独立容器中。不要为了匹配细小随机纹理，牺牲正常文档流和响应式布局。

品牌标识优先使用用户提供或获准使用的原始文件，不能默认让模型重新设计。局部裁图在分辨率、边缘和背景合格时可以直接作为素材；不合格时才用它指导重绘。重绘前明确要保留的形状与比例，不把素材生成变成每个页面的强制步骤。

图表或关系网络即使视觉复杂，只要需要准确数据或交互，就使用代码与现有库。营销插画里的非交互示意图可以作为素材，但不得把它声称为可用的数据组件。

## 还原流程

1. 沿用设计说明和现有组件，用正常布局建立页面骨架、文字、状态与响应式；完整长页先覆盖全部区块。
2. 校准字体、字号、行高、容器宽度与换行。优先可用字体，不假设生成字形能逐像素找到对应字体。
3. 检查已有素材是否够用。只对确实缺失或质量不合格的照片、插画、品牌装饰单独处理。
4. 将素材放回明确容器，设置宽高比、裁切方式和窄屏行为；文字和按钮独立实现。
5. 测试长标题、增加列表项、窄屏与关键操作状态，确认没有依赖整图背景或大量任意绝对定位。
6. 在任务允许的渲染环境中截图，先修结构与可读性，再修视觉细节。没有自动化授权时不调用浏览器，记录待验证项。

## 排版与纹理校准

文字和纹理必须作为一等还原目标，不能只看布局和素材。每次把设计稿还原成 HTML 时，都要先做一张“排版差异清单”，再修改 CSS。

必须检查这些项目：

- 字体家族：标题、正文、导航、按钮是否分别接近原图。优先沿用设计说明或项目字体；必要时选择可用的近似字体并分层指定 fallback。不要为模仿图片字形强制引入不可获得的字体。
- 字重：标题黑度、正文粗细、按钮粗细、品牌名粗细要逐项对照。AI 生成图里的标题常常比浏览器默认 `700` 更厚，正文又可能比默认 `400` 更轻。
- 字号和行高：不要只看单行高度，还要看整个文本块高度。标题行高、正文行高、按钮文字垂直居中都要单独调。
- 换行位置：标题和正文必须用文本容器宽度、`max-width`、必要的 `<br>` 或更精确的词组拆分来贴近原图。换行错了，即使字体对了，视觉重量也会明显偏。
- 字距：导航、eyebrow、logo 行标题这类小字要检查 `letter-spacing`。正文和大标题通常保持 `letter-spacing: 0`，除非原图明显有拉开。
- 颜色和抗锯齿观感：深色文字不能只用纯黑；要匹配原图里的蓝黑、灰蓝和透明度。通过实际可用的字体重量和颜色匹配，不用模糊文字阴影模拟生图噪声。
- 纹理与微妙背景：只校准已明确采用的效果。简单效果用CSS；复杂装饰作为独立素材，不为贴近随机噪声额外制造纹理。

排版校准顺序：

1. 先锁定页面宽度和主要容器宽度，因为文本换行首先由容器决定。
2. 再调字体家族、字号、字重和行高。
3. 再调文本块 `max-width`、显式换行和上下 margin。
4. 最后调颜色、字距、阴影和纹理。

排版不得只依赖像素差异热力图。像素 diff 能发现“哪里不同”，但字体和换行需要人工看 side-by-side：标题是否同样分行、每一行长度是否接近、文字黑度是否一致、正文灰度是否一致、按钮文字是否垂直居中。

## 素材生成规则

**确需生成时，大插画和 logo 分开处理。** 不要把大插画、logo、图标放在同一张素材 sheet 里。大元素会破坏网格尺度，导致自动裁切不稳定。确需生成的品牌标识与小号深色图标单独使用大尺寸白底素材，不要和 hero 图、产品图、复杂插图放在同一张板里。

**素材是否重绘由质量决定。** 先检查原始文件与可用裁图的清晰度、背景和边缘。可用就复用；需要重绘时用局部参考，随后检查裁切和透明边缘。不重绘动态图表，不将正文和业务控件合并到图片中。

**细文字和厂商 logo 优先白底，不用绿幕。** 对客户 logo row、深色文字、深色线性图标这类素材，绿幕会在抗锯齿边缘留下绿色污染，强抠图还会把细笔画吃掉。更稳的方式是生成纯白底大图，再用保守的白底转透明参数处理。这个方法适合放在白色或浅色页面背景上；如果素材要放在深色背景，优先使用真实透明输出、SVG，或重新生成深色背景专用版本。

**图标 sheet 只适合统一小图标。** 如果确实需要图标 sheet，必须让模型生成机器可切的规则网格：

```text
Machine-cuttable icon sprite sheet.
Pure white background.
Exactly 16 icons in a perfect 4 columns x 4 rows grid.
No borders, no grid lines, no labels, no text, no shadows, no decorative elements, no overlap.
Each icon is centered in its own invisible cell, same visual size, occupying only the central 45 percent of the cell, with wide white padding.
Use thin blue-gray strokes and subtle blue accents.
```

但在实际 HTML 还原里，普通图标通常还是用 SVG/icon library 更好。图标 sheet 只在用户明确想保留 AI 风格图标时使用。

## Logo / 插画素材提示词模板

Logo 单独素材：

```text
Based on the reference image, recreate only the app logo mark as a standalone asset.
Pure white background. Centered. Large size. No text, no border, no mockup, no shadow box, no extra symbols.
Preserve the logo's silhouette, proportions, blue gradient, soft depth, and brand feel.
Leave generous white padding around the logo for cropping.
```

空状态插画单独素材：

```text
Based on the reference image, recreate only the central empty-state illustration as a standalone asset.
Pure white background. Centered. Large size. No surrounding dashboard UI, no text, no buttons, no labels, no border, no grid.
Preserve the soft blue gradient, translucent panels, database cylinder, chart card, orbit line, subtle highlights, and gentle SaaS product style.
Leave generous white padding around the illustration for cropping.
```

如果需要透明 PNG，先生成纯白底或高对比纯色背景素材，再用本地工具抠图；不要让模型在同一张图里同时承担排版和透明裁切任务。

厂商 logo row 白底素材：

```text
Scene:
Pure white #ffffff canvas.

Subject:
One horizontal row of vendor logos: Amplitude, Brex, loom, Notion, Webflow, ramp.

Important details:
Large crisp vector-style dark navy marks and wordmarks, color #273142, clean kerning, consistent baseline, generous spacing, readable at web size.

Use case:
Source asset for HTML reconstruction; the white background will be removed locally.

Constraints:
Pure white background, no shadow, no glow, no texture, no gradient, no border, no labels, no grid, no watermark. Render only the logo row. Text must be legible and spelled exactly.
```

## 浏览器后验验证

把设计稿还原成 HTML 后需要验证实际渲染。以下自动化步骤仅在当前任务允许时执行；用户禁止浏览器自动化时不调用脚本，报告未验证项，不把静态检查冒充视觉通过。

1. 用独立浏览器会话打开本地 HTML，设置和原始设计稿接近的 viewport。不要临时修改用户当前窗口，也不要为了截图改页面 CSS。
2. 如果原始设计稿是固定视口图，就截取 viewport screenshot 做像素对比；同时另存一张 full-page screenshot 检查页面完整性。不要把超长整页截图压缩到设计稿高度后再判断组件位置。
3. 用 `scripts/compare_mockup.py` 对比原始设计稿和浏览器截图，输出 candidate、diff、heatmap 和 metrics。
4. 同时做分区对比：hero、导航、客户 logo 行、功能卡、指标区等关键区域都要单独 `--clip`。整页 heatmap 只能发现大问题，不能替代组件级检查。
5. 先看 heatmap 中的大块差异：首屏图片位置、标题换行、卡片间距、背景色、纹理、素材尺寸。
6. 再看组件级差异：字体家族、字重、字号、行高、卡片内部 icon 位置、标题换行、文案行宽、CTA 位置、底部插图位置、表格密度、按钮尺寸。
7. 按差异修改 HTML/CSS 或重新处理素材，然后再次截图对比。

推荐使用一体化验证脚本：

```bash
bash scripts/verify_html_mockup.sh \
  --html /path/to/page.html \
  --reference /path/to/original-mockup.png \
  --out-dir /path/to/verify-output \
  --viewport 1024x1536
```

它会通过 `agent-browser` 启动独立浏览器会话、设置 viewport、打开页面、截取 viewport 图用于对比，并额外保存 full-page 图方便检查。只有当 reference 本身就是整页长图时，才增加 `--full-page`。

单独运行对比脚本：

```bash
python3 scripts/compare_mockup.py \
  --reference /path/to/original-mockup.png \
  --candidate /path/to/browser-screenshot.png \
  --out-dir /path/to/compare-output \
  --prefix landing \
  --clip hero:0,0,1024,760 \
  --clip feature-card-1:40,910,305,365
```

调用前先确认两图像素尺寸、视口和DPR一致；脚本会自动resize，尺寸不一致时不要用其分数判断还原质量。长页高度不同时保留整页差异，改用相同坐标范围的分区检查，不能压缩整页来掩盖缺段。像素差异不替代人工判断。对于用户指出的具体区域，必须补一个 `--clip` 对比，生成单独 heatmap 后再判断。

组件级观察清单：

- 卡片：外框位置、圆角半径、padding、icon 容器位置、标题换行、说明文案行宽、CTA 位置、底部图形垂直位置。
- Hero：标题字体、字重、字号、行高、换行、按钮位置、插画大小、插画和文字的重叠关系、首屏底部露出的下一节内容。
- 导航：logo 尺寸、导航字体、导航项间距、按钮大小、左右边距。
- 数据图表：线条密度、表格行高、标签位置、空白比例。
- 纹理：页面背景柔光、卡片底色、渐变方向、雾面颗粒感、阴影扩散范围。

浏览器截图原则：

- 在有授权时使用宿主支持的独立浏览器会话，避免影响用户当前打开的页面和窗口。
- 检查用户正在看的真实页面前遵循该宿主的页面读取授权，不猜测当前页面。
- 截图前确认资源加载完成，避免拿到图片未加载时的空白截图。

## 透明素材后处理

透明输出能力取决于当前接口和模型，先检查返回文件是否真的包含透明通道。没有可用透明输出时，用纯白底或绿幕素材配合内置后处理脚本：

```bash
python3 scripts/prepare_image_asset.py input.png output.png \
  --key-color '#00ff00' \
  --key-threshold 62 \
  --feather 54 \
  --despill \
  --edge-contract 1 \
  --padding 10
```

常用策略：

- 厂商 logo、深色 wordmark、深色细线图标：优先生成纯白底大图，再用 `--alpha --threshold 248 --feather 10` 转透明，不要使用 `--edge-contract`。
- 复杂彩色插画、独立装饰图：优先让模型生成纯绿幕背景，再用 `--key-color '#00ff00'` 抠透明。
- 产品界面截图、白色卡片很多的素材：不要用白底抠图，容易误伤主体；使用绿幕。
- 复杂 hero 图如果出现绿边，增加 `--despill`，适当提高 `--key-threshold` 和 `--feather`，再轻微使用 `--edge-contract 1`。
- 如果细线、半透明阴影或柔光仍然有绿边，把素材拆成多张生成：产品面板一张、插画花束一张、装饰粒子一张，最后用 HTML/CSS 叠放。

白底厂商 logo 处理命令：

```bash
python3 scripts/prepare_image_asset.py vendor-logo-white.png vendor-logo-alpha.png \
  --alpha \
  --threshold 248 \
  --feather 10 \
  --padding 16
```

白底抠图只适合浅色页面背景。如果把这类素材放到深色底上，会看到白色抗锯齿边，这是白底混色的正常结果。深色底场景要生成深色底专用图、真实透明图，或改用矢量 SVG。

绿幕素材提示词可以这样写：

```text
Put the asset on a perfectly flat solid #00ff00 chroma-key background for background removal.
The background must be one uniform #00ff00 color with no shadows, gradients, texture, reflections, floor plane, or lighting variation.
Keep the subject fully separated from the green background with generous padding.
Do not use #00ff00 anywhere in the subject.
No semi-transparent glow, no soft shadow touching the background, no hairline strokes on the background, crisp opaque edges.
```
