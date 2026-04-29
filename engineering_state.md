# Border Ecology Box MVP 工程状态

## 1. 当前项目阶段

- 当前处于 Phase 2.6：儿童反馈提示语与感官打磨阶段。
- Phase 1 核心底层已稳定，不允许重构。
- 儿童提示语系统已接入，当前重点是实机验证文案是否清楚、不挡按钮、只在状态变化时更新。

## 2. 已稳定功能

- WebAR 地板扫描与 reticle 放置流程已跑通。
- 三个 GLB 模型已接入并可见：河豚、章鱼、刺猬。
- 模型 root 与 GLB 子模型已分层：
  - root entity 负责真实身体位置、显示状态、距离判定。
  - `a-gltf-model` 子节点负责视觉偏移、缩放、朝向与材质表现。
- 支持重新放置，不刷新页面即可回到扫描状态并在新位置放置。
- 放置后模型能固定在扫描圈附近，不再跟随 camera。
- 已加入儿童友好提示语系统：
  - 复用底部 `instruction-text`。
  - 通过 `updateHint(message)` 更新。
  - 使用 `currentHint` / `lastHintMessage` 状态锁，避免每帧重复刷新。
  - 扫描、放置、重新放置、三动物关键状态变化都会更新短句提示。
  - 已修复 safe / idle 文案接入：切换模式与放置完成后会显示当前动物安全文案。
  - 已修复章鱼提示状态：甩掉黏液后先提示退后给空间，只有真正退到安全距离后才提示距离刚刚好。
  - 已修复章鱼提示节奏：触发黏液时保护膜文案锁定约 1.8 秒，再切换到甩动提示；退回安全距离后会回到章鱼 safe 文案。

## 3. 三个动物当前状态

- 河豚 Pufferfish：
  - 靠近阈值仍为 `surfaceDistance < 0.6`。
  - 靠近时压扁，恢复自然。
  - 当前视觉 scale：
    - normal: `0.002, 0.002, 0.002`
    - pressed: `0.0022, 0.0007, 0.0022`

- 章鱼 Octopus：
  - 靠近阈值仍为 `surfaceDistance < 0.5`。
  - 触发后隐藏 3D root，显示绿色黏液 overlay。
  - DeviceMotion 解除阈值仍为三轴差值总和 `> 25`。
  - 当前视觉 scale: `0.18, 0.18, 0.18`。
  - 黏液 overlay 已改为 opacity 平滑出现 / 消失，不挡 DOM 按钮点击。

- 刺猬 Hedgehog：
  - 触发条件仍为 `surfaceDistance < 0.6 && approachVelocity > 0.3`。
  - 退后到 `surfaceDistance >= 0.8` 后等待 3 秒恢复。
  - 当前视觉 scale: `0.4, 0.4, 0.4`。
  - 朝向由 `HEDGEHOG_YAW` 控制，只作用在 `hedgehog-model` 子模型。
  - 当前 `HEDGEHOG_YAW = 180`；若方向仍不对，优先测试 `90`，再测试 `270`。
  - 红色警告已改为较柔和的粉红 / emissive 材质，不使用恐怖 wireframe。

## 4. 重要架构规则

- 不重构 Phase 1。
- 不随意改这些核心参数：
  - 地板扫描 Hit-Test UX
  - `GROUND_Y_OFFSET`
  - 三个 GLB `src`
  - 距离阈值
  - DeviceMotion 阈值
  - `approachVelocity` 阈值
  - 三个动物状态机触发条件
- AR 运行期间不要用 `appendChild` / `removeChild` 移动 3D 实体。
- 隐藏模型使用 `visible="false"` 或 root visible 状态控制。
- 不要把动物挂到 camera 下。
- 放置完成后 root 世界位置应锁定，不要每帧跟随 reticle 或 camera。
- `tick()` 内避免 DOM 查询和频繁无意义写文本；提示语用状态锁更新。
- 每次切换动物时应先隐藏全部 animal root，再只显示当前模式。

## 5. 当前 Git 工作流

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

## 6. 每次修改后的输出格式

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

## 7. 当前下一步任务

- 继续 Phase 2.6 实机验证与微调。
- 优先验证儿童提示语：
  - 初始扫描、找到地面、放置完成、重新放置时是否清楚。
  - 河豚靠近 / 恢复、章鱼黏液出现 / 甩掉、刺猬快速靠近 / 恢复时是否正确更新。
  - 章鱼甩掉黏液但仍很近时，不应直接显示“距离刚刚好”。
  - 章鱼第二次靠近时，应重新按“太近 -> 甩一甩 -> 退后 -> safe”的顺序显示。
  - 是否不挡重新放置按钮、动物切换按钮和 DEBUG 按钮。
- 若刺猬头部仍未面向用户，只改 `HEDGEHOG_YAW`，不要旋转 root。
- 后续可考虑收尾：
  - 移除或默认关闭 debug 面板。
  - 整理提示语文案。
  - 只在确认稳定后再做视觉 polish。
