# 第3章 解析与 DOM / CSSOM：HTML / CSS 解析、阻塞与 defer / async

**一句话**：`Blink` 把 `HTML` 解析成 `DOM 树`、把 `CSS` 解析成 `CSSOM`，`JS` 默认会阻塞这条解析链，靠 `defer` / `async` 才能解开。

**为什么需要它**：`DOM 树` 是后续一切的起点，`CSSOM` 阻塞 `渲染树` 的构建；而 `<script>` 默认让 `HTML` 解析暂停去执行脚本，理解这三者的阻塞关系，才能知道为什么首屏会卡、为什么样式没生效。

**核心结构**：`解析 HTML` 产出 `DOM 树`（元素/属性/文本节点树）；`解析 CSS` 产出 `CSSOM`（带层叠 cascade 与继承结果）；二者合并成 `渲染树`。`JS` 通过 `<script>` 注入，默认阻塞 `解析器`，`async` / `defer` 解除阻塞的方式不同。

**关键设计**：`CSSOM` 必须齐全才能建 `渲染树`，所以 `CSS` 是「渲染阻塞」的；`JS` 会读 `DOM`/`CSSOM`，故默认「解析器阻塞」。`defer` 下载不阻塞、推迟到文档解析完按序执行；`async` 下载不阻塞、下载完立即执行（顺序不保）。`DOMContentLoaded` 在 `DOM 树` 建完、`defer` 脚本执行后触发。

**拆解**：
### HTML 解析与 DOM 树
`Blink` 流式解析 `HTML`，自顶向下生成 `DOM 树`；遇到 `<script>`（无 async/defer）则暂停解析去执行脚本，结束再继续。
### CSS 解析与 CSSOM
`CSS`（外链/内联/style 属性）解析成 `CSSOM`，包含层叠与继承；`CSSOM` 不就绪，`渲染树` 就建不出来，故 `CSS` 阻塞渲染。
### 脚本阻塞与解除
`async` 不阻塞解析、下载完即执行（不保证顺序）；`defer` 不阻塞解析、推迟到解析结束、`DOMContentLoaded` 前按文档顺序执行。非关键脚本应两者其一。
### 就绪事件
`DOMContentLoaded` 标志 `DOM 树` 就绪且 `defer` 脚本跑完；`load` 还要等 `CSS`/图片/子资源。分清二者才能正确排初始化时机。

**今天怎么做**：首屏关键 `CSS` 内联、非关键 `JS` 用 `defer` 或 `async`、`preconnect`/`preload` 提前建连与取资源；长耗脚本拆小或用 `Web Worker`，别堵在 `解析` 与 `DOMContentLoaded` 上。

<div class="chapend" markdown="1">

**容易误解的地方**：以为 `async` 和 `defer` 一样；其实 `async` 不保顺序、随时插队执行，适合独立脚本（如统计），`defer` 保顺序、等解析完，适合有依赖的页面脚本。

**记住这一句**：`CSS` 阻塞渲染、`JS` 默认阻塞解析，`defer`/`async` 是解开这两道锁的钥匙。

</div>
