# 🌍 边界生态箱 (Border Ecology Box MVP) - 开发者核心白皮书

## 1. 产品愿景与心理学基建 (What is this project?)
本项目是一款针对 4-7 岁儿童设计的「跨文化社交边界」WebAR 体验应用（数字疫苗）。
儿童在现实空间中通过手机 AR 摄像头，与三个代表不同社交边界的“数字生命体”交互。我们不使用任何按钮控制，而是通过儿童**真实的身体移动、靠近、退后、摇晃**（具身认知 Embodied Cognition）来触发状态机，从而产生强烈的心理投射。

### 核心交互隐喻：
* **气球河豚 (Pufferfish / 柔性边界)**：隐喻“温和的抗拒”。当玩家靠近 (< 0.6m)，它会被无形的物理压力压扁（Lerp 缩放形变）。
* **粘粘章鱼 (Octopus / 高渗透边界)**：隐喻“过度热情的越界”。极近距离 (< 0.5m) 触发：3D 模型瞬间隐藏，转化为 2D 绿色的全屏幕粘液滤镜（幽闭恐惧）。解除条件：必须猛烈摇晃手机（DeviceMotion 监听，三轴加速度差值总和 > 25）将其“甩开”。
* **刺猬猫 (Hedgehog / 刚性隐形边界)**：隐喻“不可侵犯的领地”。结合距离 (< 0.6m) 与 **逼近速度 (Approach Velocity > 0.3m/s)** 进行判断。猛冲会触发红色线框警报（惩罚）；必须退后到安全距离外，并保持绝对静止等待 3 秒（延迟满足），才能解除警报。

## 2. 代码架构与引擎 (Tech Stack)
* **核心引擎**：A-Frame (1.4.2) + Three.js
* **WebXR 标准**：Hit-test (平面扫描), Local-floor (地面锚点), DOM-overlay (UI 融合层)
* **物理循环**：所有空间计算（距离、速度）全部写在 A-Frame Component 的 `tick(time, timeDelta)` 帧循环中。

## 3. 严格代码规范 (CRITICAL CODING RULES)
1.  **极简增量输出 (Diff-Only)**：绝对禁止输出和重写完整的 `index.html`！任何修改必须精准定位，只输出修改前后的代码块（Diff）。
2.  **XR DOM 安全锁**：在 AR 运行期间，绝对禁止使用 `appendChild` 或 `removeChild` 在场景树中移动 3D 实体，这会导致底层渲染崩溃。隐藏实体必须使用 `visible="false"` 或 `scale="0 0 0"`。
3.  **Tick 性能洁癖**：在 `tick` 帧循环（每秒 60 次）中，绝对禁止执行高耗能的 DOM 查询（如 `document.getElementById`）或频繁覆盖不需要更新的文本。必须引入状态锁（如 `if (state !== newState)`）来阻断多余渲染。
4.  **方向性矢量逻辑**：计算速度时，必须区分“靠近”与“后退”（如 `approachVelocity`），绝对不能使用 `Math.abs()` 造成方向性惩罚错误（惩罚用户后退）。
5.  **iOS/Android 跨端兼容**：涉及设备传感器（`DeviceMotionEvent`），必须包含苹果 iOS 13+ 特有的 `requestPermission()` 按钮点击授权弹窗，并同时包含 Android 的静默回退方案 (Fallback)。

## 4. GitHub 与测试工作流 (Workflow)
我们通过 GitHub 仓库进行版本控制，并使用自动化部署平台（如 Vercel/Netlify）进行线上测试。
* **代码修改**：Agent 在本地修改并保存代码。
* **提交准则**：每次修复单一功能后，执行 `git add .` -> `git commit -m "描述"` -> `git push`。
