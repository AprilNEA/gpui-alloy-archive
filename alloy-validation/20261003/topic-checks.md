# GPUI-001..004 migration

Upstream baseline U: `4c841aaf1c4fa613e89a5d77096523d0ff593b56`.
Upstream main inspected: `a84689073d296dfd39987bc7dd478e43ef76d83a`. All four source fixes remain absent at that commit: removed-handler cleanup absent; consumed interceptors still dispatch raw capture; TestWindow lacks accessibility callbacks; Spring uses wall time.

Validation environment: rustc 1.98.1 (48a229cea 2026-09-01), cargo 1.98.1 (797e8a9bc 2026-08-05), rustfmt 1.9.0-stable; native macOS aarch64; original Cupertino devenv shell; fixed upstream Cargo.lock.

Each topic starts at U. README human review marker remains uncommitted in the assigned worktree. No pushes or PRs.

## GPUI-004

Branch: `fix/gpui-spring-clock`. Commit: `80222ec015a7ba381f3cd6bb95b35bc0476b15bf`. Status: committed.
Checks: rustfmt --edition 2024 --check on modified file passed; cargo test -p gpui --lib --release --locked: 347 passed, 0 failed; ./script/clippy -p gpui --locked passed (release, all targets, all features); git diff --check passed. No native rendering behavior changed. Cargo reports existing block 0.1.6 future-incompatibility warning. Optional cargo-shear is not installed; script exits successfully before optional checks.
Source author: AprilNEA <github@sku.moe>, implementation from Cupertino `ebba53ae60327ac0050d8f0be543642d90996dbb`.
Before-fix evidence: parent baseline full `cargo test -p gpui --lib --release --locked` had 345 passed / 1 failed, `elements::animation::tests::test_spring_animation_preserves_velocity_when_retargeted` at animation.rs:742.

## GPUI-001

Branch: `fix/gpui-cancel-pending-activation`. Commit: `05dc2cf96a652c23ab1a54bfc5ba7e5f762c66e5`. Status: committed.
Checks: rustfmt --edition 2024 --check passed; cargo test -p gpui --lib --release --locked elements::div::tests:: — 27 passed, 0 failed; ./script/clippy -p gpui --locked passed; git diff --check passed. Two initial build attempts failed because shared target reused the rendering topic gpui_apple build script and requested absent continuous_corners.metal; cargo clean -p gpui_apple --release removed only the stale package artifacts, then the original check commands passed. No source workaround was introduced. Full suite was not rerun on this independent topic; its unchanged Spring source has the known U failure. GPUI-004 is not included.
Source author: AprilNEA <github@sku.moe>, keyboard fix from Cupertino `ef5d17f00df73cab140b071ff23ee220738b6aff`; pointer fix from Cupertino c79bb253dd456877b61e1503ec4f2c549838a86e, matching the frozen source.

## GPUI-002

Branch: `fix/gpui-interceptor-propagation`. Commit: `46d1fe8255b73a46413d4fe579ec3504e7f38a66`. Status: committed.
Checks: rustfmt --edition 2024 --check on all modified files passed; cargo test -p gpui --lib --release --locked key_dispatch::tests:: — 23 passed, 0 failed; ./script/clippy -p gpui --locked passed; git diff --check passed. Full suite was not rerun on this independent topic; its unchanged Spring source has the known U failure. GPUI-004 is not included.
Source: Cupertino c79bb253dd456877b61e1503ec4f2c549838a86e, matching the frozen vendor; contract documentation now includes raw keyboard handlers.

## GPUI-003

Branch: `test/gpui-accessibility-window`. Commit: `e334e7b64830121436cbffcca83521fd8c7173ef`. Status: committed.
Checks: rustfmt --edition 2024 --check on both modified files passed; cargo test -p gpui --lib --release --locked platform_accessibility_activation_and_actions_reach_the_rendered_view: 1 passed; ./script/clippy -p gpui --locked passed; git diff --check passed. Includes public TestAppContext::test_window accessor. Full suite was not rerun on this independent topic; its unchanged Spring source has the known U failure. GPUI-004 is not included. Native screen reader behavior remains unverified.
Source author: AprilNEA <github@sku.moe>, Cupertino `1eb328bca34ca7c51426e9472110d04111aee385`. Test is transferred from Cupertino's pure GPUI accessibility test.


## Final worktree state

All four source topics are committed independently on U; each commit carries GPUI-Patch and source provenance. Worktree is on fix/gpui-interceptor-propagation with only the required README review marker uncommitted. No Cargo processes remain. No pushes or PRs were made.

## Additional read-only review

Reviewed GPUI-005 commit e9a82d67908b5d4252538d1731e7a43df874b1ca against frozen vendor. No missing inert source hunks or conflict with GPUI-001 was found. GPUI-001 request-layout cleanup handles absent listeners; GPUI-005 prepaint cleanup independently handles the inert boundary. Repeated clearing is idempotent. Focus-reveal hunk belongs to GPUI-006.

# GPUI-005 / GPUI-006 migration

Source: AprilNEA/gpui-cupertino commit `c79bb253dd456877b61e1503ec4f2c549838a86e`. Frozen source files are under `/tmp/gpui-alloy-migration.wjfUV4/source/vendor`, with regression scenarios adapted from the adjacent frozen Cupertino source. Upstream baseline U: `4c841aaf1c4fa613e89a5d77096523d0ff593b56`.

## GPUI-005

- Branch: `feat/gpui-inert`.
- Commit: `e9a82d67908b5d4252538d1731e7a43df874b1ca`.
- Base: U; no other patch dependency, including GPUI-003.
- Production scope: transparent `inert` element; scoped Window registration suppression; deferred draw state; cached-view invalidation; a11y and Div activation cleanup.
- Tests: new pure GPUI regressions cover deferred layout/paint preservation, input/focus/a11y exclusion, cached subtree transitions, canceled keyboard and pointer presses, and platform text-input registration transitions.
- Passed: `cargo fmt -p gpui -- --check`; `./script/clippy -p gpui --locked` (release/all-targets/all-features); `cargo test -p gpui --lib --release --all-features --locked inert` (4); the same test command filtered by `input::tests` (2); the same test command filtered by `elements::deferred::tests` (1).
- Root README manual-review marker remains present and uncommitted; the topic commit includes GPUI README usage docs only.

## GPUI-006

- Branch: `feat/gpui-autoscroll-on-focus`.
- Stacked base: GPUI-005 commit `e9a82d67908b5d4252538d1731e7a43df874b1ca`.
- Commit: `32bc7317a4e7a6bcc4b4d18121608c7d270ff451`.
- Passed: `cargo fmt -p gpui -- --check`; `./script/clippy -p gpui --locked`; `cargo test -p gpui --lib --release --all-features --locked -- scroll_focus_tests elements::inert::tests input::tests elements::deferred::tests` (11 passed).
- The first Clippy run identified an unused macro import in the adapted test. Removing that import resolved the error; no lint suppression or weakened assertion was added.
- The release cache for `gpui_apple` was cleaned before validation after the worktree switch.
- Trailer: `GPUI-Patch: GPUI-006`. The earlier GPUI-005 commit retains its original `Patch-ID` trailer; integrated history was not rewritten.
- Production scope: `Div::autoscroll_on_focus`, scoped focus-reveal target, nested minimum-displacement offsets, focus generation consumption after drawing, transaction rollback.
- Tests cover cached targets, Tab navigation, free wheel scrolling after reveal, nested viewports, inert/deferred descendants, oversized targets and offset clamping.

## Scope and validation limits

The existing `upstream/main` ref was inspected at `a84689073d296dfd39987bc7dd478e43ef76d83a`. No matching `inert`, `autoscroll_on_focus`, focus-reveal, or focus-scroll implementation was found in GPUI. No remote ref was changed.

No GPUI-001 handler-removal patch, GPUI-002 interceptor patch, GPUI-003 platform helper patch, rendering patch, consumer change, remote push, or PR creation is included. Native desktop accessibility and IME behavior were not manually exercised. The parent recorded the unrelated baseline Spring-clock test failure; GPUI-004 addresses that issue and is intentionally absent from these topics.

# GPUI 渲染迁移记录

工作区：`/tmp/gpui-alloy-migration.wjfUV4/render`。上游基线 `U = 4c841aaf1c4fa613e89a5d77096523d0ff593b56`。未推送、未创建 PR。根 README 人工审阅标记保留在工作区，未加入纯 GPUI topic 提交。

## 来源与拆分

原作者为 `AprilNEA <github@sku.moe>`。冻结的渲染源码逐文件核对，与 Cupertino `be245ff8b0d15db9e9b4e90b64ffd90fcd2788b6` 一致。window.rs 使用相应历史版本提取渲染 API，排除后来加入的输入、inert 和 focus 修改。最终渲染组合保留冻结实现；不带 vendor workspace 裁剪或 shaders.metal 无关空白清理。

- GPUI-007：`feat/gpui-ordered-backdrop`；从 `U` 独立建立。来源为 `9c8b4e23c299006e2440eada4043dc1d04c2a750` 与 `ebba53ae60327ac0050d8f0be543642d90996dbb`。包括全局材质顺序边界、打开图层重定位、缓存重放、边缘裁剪、Metal 截图与 Gaussian 合成、WGPU 显式拒绝和 window 测试访问器。`libm = 0.2.16` 属于 Gaussian 核；lockfile 增加 gpui_apple → libm。
- GPUI-008：`feat/gpui-continuous-quad`；依赖 GPUI-007。来源为 `be245ff8b0d15db9e9b4e90b64ffd90fcd2788b6`。包括独立 continuous quad batch、未改变的普通 Quad GPU 布局、共享 continuous corner Metal 函数和 build.rs 拼接路径。Backdrop 同步使用连续轮廓；祖先 opacity 在线性颜色空间组合，几何边缘 coverage 按编码后的窗口值混合。保留对应独立数值参照断言更新。
- GPUI-009：`local/gpui-inactive-clear`；依赖 GPUI-007、GPUI-008。来源为 `be245ff8b0d15db9e9b4e90b64ffd90fcd2788b6`。包括独立 Clear scene 记录、encoded-RGB mip 过滤、Apple GPU 路径、实时 layer/display ColorSync 参数转换、Clear 访问器和公开 ColorSync CPU 参照检查。

## 检查状态

GPUI-007 的 5 项 scene 顺序／重放回归，以及 11 项 Metal backdrop／Gaussian／颜色合成／覆盖率／折射参照检查已通过。`./script/clippy -p gpui -p gpui_apple -p gpui_wgpu --locked` 和 targeted cargo fmt 检查通过。Metal integration targets 显式要求 test-support；cargo metadata 核对通过。提交为 `2387142b99ef39787f872e1d3a5925b259627b1d`。

GPUI-008 的提交为 `a09b1d01f967f05b6c18fdb93d70dd625b17c07d`。5 项 scene 与 14 项 Metal 检查通过；其中 3 项 continuous corner 检查另在未启用 runtime_shaders 的模式下通过，覆盖 build.rs 的预编译 Metal 路径。对应 `./script/clippy` 与 targeted cargo fmt 检查通过。首轮 native 测试发现共享 target 中旧 build-script 产物与新源码失配，生成的 stitched shader 缺少共享圆角函数定义。执行 `cargo clean -p gpui_apple --release` 后完整重跑通过，没有修改源码或测试断言；首轮日志保存在 `render-008-metal-stale-build.log`。

GPUI-009 的提交为 `ae2f53dcf6e8a25e933c695fbe7bc30079a04832`。5 项 scene 与 17 项 Metal 检查通过，其中包含 2 项 Clear 固定样本／clip 检查和 1 项 public ColorSync CPU/GPU 对照检查。对应 `./script/clippy`、targeted cargo fmt 和全部 6 个 native test target 的 test-support 配置核对通过。没有运行中的 Cargo 检查，共享 target 已交回父任务。

最终忽略 rustfmt 空白后，16 个渲染实现文件与冻结 vendor 源码完全一致；window.rs 与 `be245ff8b0d15db9e9b4e90b64ffd90fcd2788b6` 的仅渲染历史版本一致。核对脚本为 `check-render-source.py`，不包括额外的测试 target 配置和 README 说明。三条分支依次相接，父任务可按 GPUI-007 → GPUI-008 → GPUI-009 顺序集成；三个提交均未签名、保留原作者和 Source-Commit 记录。

验收命令记录：

```sh
# GPUI-007
cargo test -p gpui --test ordered_backdrop --release --all-features
cargo test -p gpui_apple --release --all-features --locked --test backdrop --test numeric_reference --test optical_reference

# GPUI-008
cargo test -p gpui --test ordered_backdrop --release --all-features --locked
cargo test -p gpui_apple --release --all-features --locked --test backdrop --test numeric_reference --test optical_reference --test continuous_corners
cargo test -p gpui_apple --release --features test-support --locked --test continuous_corners

# GPUI-009
cargo test -p gpui --test ordered_backdrop --release --all-features --locked
cargo test -p gpui_apple --release --all-features --locked --test backdrop --test numeric_reference --test optical_reference --test continuous_corners --test clear --test color_management

# Each topic passed these checks before its commit.
./script/clippy -p gpui -p gpui_apple -p gpui_wgpu --locked
cargo fmt --package gpui --package gpui_apple --package gpui_wgpu --check
```

对应完整日志分别保存在 `render-007-*.log`、`render-008-*.log` 和 `render-009-*.log`。Clippy 脚本按仓库实现运行 release/all-targets/all-features/deny warnings；本机未安装可选 cargo-shear，脚本因此跳过该可选步骤并成功退出。没有安装新工具。

运行环境通过原 Cupertino 的 `devenv shell -- bash /tmp/...` 提供。所有 Rust 检查使用共享 `CARGO_TARGET_DIR=/tmp/gpui-alloy-migration.wjfUV4/target`；测试使用 release profile。父任务已确认上游完整 GPUI baseline 中 Spring wall-clock 回归失败，该问题属于独立 GPUI-004，本主题不修改 Spring。

## 平台与验证边界

Backdrop 仅支持原生 macOS Metal 中的早先不透明 SDR 窗口内容，不采样桌面，不支持透明窗口或 HDR/EDR。每个效果保留完整窗口截图资源分配。Continuous quad 仅接受相等圆角、等宽 solid 边框。Inactive Clear 要求 Apple GPU、Metal 3.1 shader 能力、不透明 SDR 目标及可参数化的 RGB 色彩配置。

Metal 像素检查不代表真实窗口多显示器、混合 DPI、实时显示器迁移或完整窗口合成验收。Native 样本沿用原仓已记录的 macOS 26.4（25E246）固定数据及 PNG SHA-256，本轮未重跑原研究记录的完整 24 次抓取／104 个区域验收。本轮不自动启用功能、不修改原 Cupertino manifest／lockfile／vendor，也不宣称获得上游维护者批准。

# GPUI-010 hidden popup lifecycle correction

- Branch: `local/gpui-010-native-popup`.
- Fix commit: `225e82553c278211ff44bff3bca5d0f7d1fefb3f`.
- Parent topic commit: `f00547a0b37e8a6b56cf18652377d8053b382d02`.
- Upstream base: `4c841aaf1c4fa613e89a5d77096523d0ff593b56`.
- Trailer: `GPUI-Patch: GPUI-010`.
- The correction was authored during the migration review. The original topic records source PR `zed-industries/zed#64353`, source commit `619d4b2b3da3040e61c027a394aa5417123e92d0`, and source base `6872711ff860c18e3d44a0171105273b6717ee15`.

## Behavior

A grabbing popup created with `show: false` must survive application deactivation before explicit activation. The popup must still close with its parent. After explicit activation, application deactivation must close the popup even if AppKit has already hidden or detached the native panel.

`MacPopup::show_requested` records the initial `WindowOptions.show` value and becomes true in `PlatformWindow::activate`. Application-deactivation and global-mouse callbacks consult that state. Parent-close and Escape paths retain their existing behavior. The GPUI README describes hidden popup behavior.

Native visibility and parent attachment cannot represent this state. Apple documents that `NSPanel` hides on deactivation by default and that `orderOut` detaches a child window. See [hidesOnDeactivate](https://developer.apple.com/documentation/appkit/nswindow/hidesondeactivate?changes=_4) and [parentWindow](https://developer.apple.com/documentation/appkit/nswindow/parent?language=objc).

## Verification

The new native regression first creates a hidden grabbing popup. The test posts `NSApplicationDidResignActiveNotification` through the native `NSNotificationCenter` and verifies that the popup survives. The test explicitly activates the popup, orders the panel out, posts the same notification, and verifies that the popup closes. Existing native checks still cover hidden parent-close cleanup, consumed and modified Escape, unhandled Escape, and passive popups.

Before applying the correction, the new regression failed on the parent topic commit with: `application deactivation closed a popup before its first activation`. The failure was produced by `cargo test -p gpui_macos --test native_popup --release --features test-support --locked`.

After applying the correction, all checks passed:

- `rustfmt --edition 2024 --check crates/gpui_macos/src/window.rs crates/gpui_macos/tests/native_popup.rs`.
- `git diff --check`.
- `./script/clippy -p gpui_macos --locked`.
- `cargo test -p gpui_macos --lib --test native_popup --release --features test-support --locked`: 20 macOS unit tests and the complete native popup regression harness.

Commands ran through the existing Cupertino devenv shell. `CARGO_TARGET_DIR` was `/tmp/gpui-alloy-migration.wjfUV4/target`. `cargo clean -p gpui_apple --release` ran before the first build after switching worktrees. The existing `block 0.1.6` future-compatibility warning remained; no suppression was added.

The regression exercises the native deactivation observer. The global mouse callback uses the same state gate, but no external-application mouse event was synthesized. The popup example test had passed for the parent topic and was not rerun for this lifecycle correction.

No original PR branch, integration branch, consumer source, remote branch, or PR was changed. The root README review marker remains present and uncommitted. The commit is unsigned. No Cargo process remains active.

# Integration 生产源码完整性核对

2026-10-03，只读检查 `/tmp/gpui-alloy-migration.wjfUV4/integration` 与同目录 `source/vendor`。首次检查开始时 integration 为 `d789e210b8`，期间父任务集成了 GPUI-001；该轮复查 HEAD 为 `16ea6dbe73d2178cca758a51e1c78b8ff760938a`。GPUI-002／006 集成后，在 `38f3cb8a160f935981b1dfb8eaa15f32cf27e7f6` 完成 `div.rs`／`window.rs` 最终复查，并检查 GPUI-002 的控制流和测试保留。全程未运行 Cargo，未修改仓库文件。

## 结论

冻结 `crates/` 与 `tooling/` 下共核对 266 个生产代码或构建输入文件，覆盖 Rust、Metal、WGSL、HLSL、C、JS、资源定义及构建脚本。排除测试、文档、示例、fixture 和构建产物；忽略空白后，没有缺失路径。最终 GPUI-001 至 GPUI-009 的生产实现均已完整保留，未发现 GPUI-010／011 额外功能之外的生产差异。

全部渲染生产文件、GPUI-003 helper、GPUI-004 时钟修复、GPUI-005 inert 注册抑制与缓存处理均保留。另核对了 `app/test_context.rs` 与 `platform/test/window.rs` 的 GPUI-003 helper：实现保留，后者仅新增 inline regression test。含 inline test 的差异文件逐个人工核对了生产部分，没有仅因测试不同而漏判生产差异。

## 差异分类

| 文件 | 剩余差异 |
| --- | --- |
| `crates/gpui/src/elements/div.rs` | 最终生产部分与冻结源一致。剩余差异仅为 GPUI-001 的 inline tests 及 GPUI-006 的 `#[cfg(test)]` 测试模块声明。 |
| `crates/gpui/src/window.rs` | 最终整个文件忽略空白后与冻结源一致；没有渲染、inert、interceptor 或 focus reveal 实现遗漏。 |
| `crates/gpui/src/app.rs` | GPUI-002 仅扩充 interceptor 停止 raw keyboard event handlers 的契约文档。 |
| `crates/gpui/src/key_dispatch.rs` | GPUI-002 仅新增 raw capture/bubble 拦截回归测试。 |
| `crates/gpui/src/elements/animation.rs` | 仅新增 scheduler clock 回归测试。 |
| `crates/gpui/src/elements/inert.rs` | 仅新增 inline regression tests。 |
| `crates/gpui/src/input.rs` | 仅修改测试 view 并增加 inert text-input 注册切换测试。 |
| `crates/gpui/src/platform/popup.rs` | 仅 GPUI-010 的跨平台 grab 行为文档。 |
| `crates/gpui/src/platform.rs` | 仅 GPUI-011 的无 titlebar popup 装饰文档。 |
| `crates/gpui_macos/src/platform.rs` | GPUI-010：移除旧 `AnchoredPopup` 拒绝分支，传递 native window 创建错误。 |
| `crates/gpui_macos/src/window.rs` | GPUI-010：native popup 生命周期、父子窗口、grab/focus、resize、Escape、close；GPUI-011：无 titlebar PopUp 使用 borderless 并关闭 native shadow。 |

Integration 额外的 `crates/gpui_macos/src/window/popup.rs` 属于 GPUI-010，不是冻结源遗漏。除冻结源本身的新增代码外，没有发现其他额外生产代码主题。

## Manifest 与锁文件

- `crates/gpui_apple/Cargo.toml` 的生产依赖与冻结源一致，包括 `libm = "0.2.16"`；差异仅是 6 个 native test target 的 `test-support` 条件。
- `crates/gpui_macos/Cargo.toml` 的额外生产 feature 为 GPUI-010 所需的 `NSEvent`、`NSOperation` 和 `block2`；其余差异为 native test target 与 `NSGraphicsContext` 测试依赖。
- 其他导出包 manifest 与冻结源相同。根 `Cargo.toml` 保留完整上游 workspace，与 vendor 的裁剪 workspace 不应逐字相同；根 manifest 相对 U 没有修改。
- `Cargo.lock` 相对 U 仅增加 gpui_apple → libm 依赖边，没有包版本变动。

## GPUI-002／006 最终复查

1. GPUI-002 集成提交为 `014800185f549cd48fa637bb08e10fa29ae742ee`。`dispatch_keystroke_interceptors` 后，`!cx.propagate_event` 分支立即返回，不调用 `finish_dispatch_key_event`，也不会进入后续 pending input 与 dispatch tree 处理。
2. `test_interceptor_stops_raw_capture_and_bubble_handlers` 保留在 `key_dispatch.rs`。测试先消费 `f1` 并断言 capture/bubble 均为 0，再关闭消费并断言二者均为 1；覆盖停止与正常传播两条路径。既有 `test_consumed_modifier_preserves_pending_input` 仍保留。该文件和契约文档相对 GPUI-002 集成提交无后续改动。
3. GPUI-006 集成提交为 `38f3cb8a160f935981b1dfb8eaa15f32cf27e7f6`。`div.rs` 在首个 `#[cfg(test)] mod tests` 之前的全部生产文本，与冻结源忽略空白后完全一致；`window.rs` 整体同样一致。因此 GPUI-001、005、007、008、009 的相邻修改均保留。
4. `div/scroll_focus_tests.rs` 模块仍在 `#[cfg(test)]` 下连接，保留 5 项测试，覆盖 cached/Tab、自由 wheel scrolling、嵌套滚动、inert、deferred、大目标及 offset clamp。

GPUI-010 隐藏窗口后续修复尚未进入此次审阅的 HEAD，不属于本报告结论。GPUI-010／011 的 macOS 额外实现继续按各主题自身测试和平台限制验收，不要求与冻结源相同。多显示器、混合 DPI 和完整窗口合成不在本次源码完整性核对结论内。此次最终复查只确认源码与测试保留，不新增运行通过的声明。
