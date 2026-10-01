# PROJECT.md — 像素云朵小狗掌机 · AI 协作规范

## 一、项目概述
- 复古掌机风格电子宠物网页游戏（PIXELPET-5 Cinnamoroll），160×128 canvas，竖屏手机优先。
- 技术栈：原生 JS（ES Modules）+ Canvas 2D + Web Audio + LocalStorage，零依赖零构建。
- 现状：main 分支单文件 index.html（约 1000 行）可运行；正在做 **5 文件模块化重构**。
- 运行方式：需本地 HTTP 服务器（VSCode Live Server 等），`file://` 直开会被 CORS 拦截。

## 二、仓库与读取方式
- 仓库：[https://github.com/Cicelina/pet](https://github.com/Cicelina/pet)（公开）
- 分支：`main` = 稳定版；`refactor/modules` = 拆分工作分支（重构启动时创建），联调通过后合回 main。
- **AI 读代码规则**：
  1. 有网页读取能力时：直接读仓库，源码走 jsDelivr，例如 `https://cdn.jsdelivr.net/gh/Cicelina/pet@main/index.html`（把 main 换成分支名、文件名换路径即可）。`.js`/`.txt` 可原文直读；`.html` 会被渲染剥掉标签，只能读到可见文本，骨架请用户提供。
  2. 单次读取视口约 100~150 行。定点修改先用搜索定位目标函数，只读附近窗口，不必通读全文。
  3. 动手前先核对本文件「八、进度」与仓库实际状态，与用户本地进度对齐。
  4. 无网页读取能力时：请用户按「【js/路径】+ 完整内容」格式贴文件。

## 三、目标架构（5 个 JS 文件）

~~~
pet/
├── index.html      # ~60 行   DOM 骨架
├── css/style.css   # ~150 行  全部样式
├── PROJECT.md      # 本文件
└── js/
    ├── core.js     # ~200 行  数据总表(食物/台词/场景/OUTINGS/NAV/帧表FR/游戏数值)
    │                        + S存档 + R运行时状态 + save/load/tick
    ├── engine.js   # ~250 行  utils + 音频(AudioFX/BGM) + emoji像素渲染 —— 封版，写完不动
    ├── screen.js   # ~250 行  画布总成：场景painter + 宠物绘制 + 特效 + HUD
    ├── world.js    # ~300 行  全部玩法：喂食/洗澡/睡觉/治疗/抚摸 + 外出/超市/三小游戏
    │                        + AI聊天 + 控制台（文件内用横幅注释分区）
    └── main.js     # ~80 行   启动 + 主循环 + 输入分发
~~~

依赖单向，禁止循环引用：`core → engine → screen → world → main`

## 四、文件生长规则（防止将来返工）
- 单文件 **< 500 行** → 不拆。
- **> 500 行，或一个文件内出现 ≥ 2 个独立变化原因** → 拆分。
- 拆分按「改动原因」切，不按技术层切。预期第一刀：world.js 涨过 500 行时，把游戏厅三小游戏拆成 `games.js`。
- world.js 内部用横幅注释分区：`/* ============ 音游 ============ */`，方便人与 AI 定位。

## 五、核心开发原则
1. **一个改动原因 = 尽量一个落点文件**。写任何新代码前先问：这个需求下次再变时，应该只打开哪个文件？
2. **数据驱动**：菜单/目的地/食物/按钮等界面列表一律遍历 core.js 数据表生成，禁止在逻辑里硬编码条目。加一种食物 = core 加一行，界面自动出现。
3. **功能自包含**：某功能的菜单 DOM、流程逻辑、进入/退出钩子集中在一处；新增目的地（如咖啡店）= core 加条目 + 对应分区加逻辑，调度代码保持封版。
4. **画布绘制集中在 screen.js**：各场景为独立 painter 函数，互不穿透；屏内像素物件预渲染缓存，主循环零逐像素写入。
5. **图标一律 emoji 像素渲染**（engine.js 的 mkEmo/dEmo），不再手写点阵。
6. **角色 = 4×4 精灵图**：帧位映射集中在 core.js 的 FR 表；素材 URL 可导入替换并存 LocalStorage。

## 六、功能约定（重构必须保留，不可丢）
- 存档：LocalStorage 单 key JSON；含离线结算 applyOffline；清档需三连确认。
- 状态四维 hunger/clean/happy/energy，周期 tick 衰减；过低触发 hot 按钮提示与需求气泡。
- 成长：蛋(点3下孵化)→幼崽→少年→成年，GROW_TH 阈值；成年解锁飞行。
- 场景 garden/room/beach/snow 按阶段解锁；昼夜 day/dusk/night 随现实时间，可锁定。
- 外出：公园（抓蝴蝶）/超市（购物+背包12格）/游戏厅（音游/跳跳乐/泡泡 + 娃娃机15币/次）。
- 游戏细节：音游 A/S/D 或点击轨道、FEVER×2；跳跳乐二段跳/护盾/磁铁/双倍。
- 洗澡搓泡沫交互；睡觉 ZZZ 粒子+精力恢复；治疗闪光+治愈音。
- AI 聊天：硅基流动 API（Qwen2.5-7B），Key 仅存本机；语音输入用 SpeechRecognition。
- 控制台：快捷按钮 ⭐阶段 🌗昼夜 🍚饱食 ⚡精力；命令 stage/day/dusk/night/auto/hunger/energy/happy/clean/coins/all/cure/sick/name/scene/bag/best/poop/egg/js/clear。
- 音频：AudioContext 首次交互时初始化；BGM 昼夜双曲谱；场景切换音效齐全。

## 七、协作流程与输出规范
1. 用户提需求（可不含代码）→ AI 读仓库定位改动点，先用一句话说明本次动哪些文件。
2. 按文件输出改动：**每个文件一个完整代码块，首行注明路径**，用户整份覆盖保存。小改动可给「定位锚点 + 替换片段」，但须写明插入位置。
3. 输出末尾附「改动清单」：动了哪些文件、各是什么性质（新建/重写/+N 行）。未列出的文件即无需动。
4. 数值调整优先落在 core.js 数据表并给建议值，不散落魔法数。
5. 用户保存后提交：`git add -A && git commit -m "feat/refactor: 简述" && git push`（可攒几个文件一起推）。
6. 大型重构按依赖顺序逐文件交付，每个确认后再发下一个，不一次性倾倒全项目。

## 八、进度追踪（每次会话先核对，交付后更新并 push）
- [x] 单文件原型（main 分支 index.html，~1000 行）
- [ ] js/core.js
- [ ] js/engine.js
- [ ] js/screen.js
- [ ] js/world.js
- [ ] js/main.js + index.html + css 迁移收尾
- [ ] 联调通过，refactor/modules 合回 main
- **备注**：2026-10-02 最近交付：PROJECT.md 本身（删除外层代码围栏与尾部操作步骤、清理引用残留、由 project.md 更名）。卡点：index.html 实际行数与功能细节未在线核实（html 走渲染通道只能读到可见文本），启动拆分前需用户提供完整源码；refactor/modules 分支尚未创建。
