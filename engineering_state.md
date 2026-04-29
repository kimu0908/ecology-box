# Border Ecology Box MVP 工程状态

## 1. 当前项目阶段

- 当前已进入 Phase 3：UI / 视觉包装 / 启动画面优化。
- Phase 1 核心底层稳定，不允许重构。
- Phase 2 核心交互已基本落地，后续 UI 优化阶段不要随便修改三个角色状态机。

## 2. 已稳定功能

- WebAR 地面扫描、放置、重新放置功能已经稳定。
- 三个 GLB 模型都能正常显示和放置。
- 模型不会跟随 camera 移动。
- 地面高度 offset 已校准。
- root entity 与 GLB 子模型已分层：
  - root entity 负责真实身体位置、显示状态、距离判定。
  - `a-gltf-model` 子节点负责视觉偏移、缩放、朝向与材质表现。
- 儿童提示语系统已接入，提示语会跟随关键状态变化更新。

## 3. 三个动物当前状态

### 河豚 Pufferfish

- 可见。
- 距离 `< 0.6m` 时触发压扁。
- 退后后恢复。
- 压扁与恢复已能表达“靠太近会挤到对方”的教育含义。
- 当前视觉 scale：
  - normal: `0.002, 0.002, 0.002`
  - pressed: `0.0022, 0.0007, 0.0022`

### 章鱼 Octopus

- 可见。
- 距离 `< 0.5m` 时触发黏液。
- 触发后必须先退到 `0.8m` 外。
- 退到 `0.8m` 外后，才允许甩动手机解除黏液。
- 在 `0.8m` 内甩动不会解除黏液。
- DeviceMotion 甩动阈值仍为三轴差值总和 `> 25`。
- 当前逻辑用于表达“先给对方空间，才可以修复互动”。
- 当前视觉 scale: `0.18, 0.18, 0.18`。

### 刺猬 Hedgehog

- 可见。
- 快速靠近时，通过 `approachVelocity > 0.3` 触发警戒。
- 慢慢靠近不会触发警戒。
- 慢慢靠近成功时会出现正向提示。
- 退回安全距离后，可以重复触发慢慢靠近的正向提示。
- 当前逻辑用于表达“慢慢接近、等待和保持距离是一种友好方式”。
- 当前视觉 scale: `0.4, 0.4, 0.4`。
- 朝向由 `HEDGEHOG_YAW` 控制，只作用在 `hedgehog-model` 子模型。

## 4. 重要架构规则

- 不重构 Phase 1。
- 后续 UI 优化阶段不要随便修改：
  - Hit-Test 扫描逻辑
  - 重新放置逻辑
  - `GROUND_Y_OFFSET`
  - 三个模型 `src`
  - 河豚距离阈值
  - 章鱼 `0.5m / 0.8m` 逻辑
  - 章鱼 DeviceMotion 甩动阈值
  - 刺猬 `approachVelocity` 阈值
  - 三个角色核心状态机
- AR 运行期间不要用 `appendChild` / `removeChild` 移动 3D 实体。
- 隐藏模型使用 root `visible` 状态控制。
- 不要把动物挂到 camera 下。
- 放置完成后 root 世界位置应锁定，不要每帧跟随 reticle 或 camera。
- `tick()` 内避免 DOM 查询和频繁无意义写文本；提示语用状态锁更新。

## 5. Phase 3 下一步任务

### UI 清理

- 隐藏或关闭 debug 面板。
- 优化提示文字框。
- 优化按钮样式。
- 统一界面视觉风格。
- 不要再动核心交互状态机。

### AR 进入流程

- 尽量去掉不需要的 VR 入口。
- 保留 AR 体验入口。
- 设计一个类似游戏初始画面的欢迎页。

### 启动页

- 标题：邊界生態箱 / The Boundary Terrarium
- 简短说明：探索小动物喜欢的距离，学习慢慢靠近与尊重空间。
- 一个“开始体验”按钮。
- 可使用一张 GPT Image 生成的主视觉图作为背景或封面。

### 后续可选

- 轻量 idle 动画。
- 提示文字淡入淡出。
- 按钮 hover / active 微动效。
- 暂时不做复杂骨骼动画或开场影片。

## 6. 当前 Git 工作流

- GitHub 仓库：`https://github.com/kimu0908/ecology-box`
- 本地项目目录：`C:\Users\user\Desktop\Border Ecology Box MVP`
- 只提交本次实际修改的必要文件，禁止 `git add .`。
- 不提交本地杂项：
  - `cloudflared.exe`
  - `lhr.txt`
  - `pinggy.txt`
  - `compress_glb.py`
  - `.vscode/`
- Codex 当前环境通常无法写 `.git/index.lock`，也可能没有 GitHub push 凭证。
- 如果 Codex 不能 commit / push，必须给用户完整 PowerShell 命令。

## 7. 每次修改后的输出格式

每次代码修改后，回复应包含：

- 改了哪些文件。
- 本次最小 diff 摘要。
- 自检结果。
- 是否影响 Phase 1：
  - 地板扫描
  - 三角色距离触发
  - 章鱼 DeviceMotion
  - 刺猬 approachVelocity
- commit message。
- 如果无法 push，给完整 PowerShell 命令：

```powershell
cd "C:\Users\user\Desktop\Border Ecology Box MVP"
git status
git add <本次实际修改的文件>
git commit -m "<合适的提交信息>"
git push origin main
```

- push 后手机测试点。
