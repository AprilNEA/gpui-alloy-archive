# GPUI Alloy 维护规范与补丁台账

GPUI Alloy 维护一份跟踪 Zed 的 GPUI 下游版本，同时把可通用的修改独立提交给上游。允许保留自有补丁，但每个补丁必须有来源、用途、依赖、验证结果和退出条件。消费方必须使用可重现的固定快照。

本规范覆盖分支、补丁台账、上游 PR、同步、验证、分发和回退。仓库初始化不代表补丁已经迁移或快照可以消费。初始化状态、首次推送步骤和待迁移补丁见文末台账。

## 维护边界

- 在 [AprilNEA/gpui-alloy](https://github.com/AprilNEA/gpui-alloy) 中维护下游集成和本台账。保留完整 Zed 仓库与提交历史，沿用上游 crate 名称和目录。Alloy 是独立 GitHub 仓库，不属于 Zed 的 GitHub fork 网络。
- 在现有 [AprilNEA/zed](https://github.com/AprilNEA/zed) fork 中保留上游 PR 的 topic 分支。从记录的 Zed 基线创建新 topic，通过该 fork 向 Zed 提 PR。已有两个 PR 保持原位置。
- 在 [AprilNEA/gpui-cupertino](https://github.com/AprilNEA/gpui-cupertino) 中维护 Cupertino 组件、vendor 导入记录和消费方验收证据。初期继续使用现有 `vendor/`，不同时引入另一种消费渠道。
- 将 Zed 作为核心 API 和平台行为的上游。将 Cupertino 专用材料和尚未被上游接受的能力作为明确登记的自有补丁。
- 不要求所有自有补丁都提交上游。拒绝、延后或放弃上游提交时，记录理由和后续维护责任。
- 由一名集成维护人负责每次同步和快照验收。其他贡献者通过独立主题提交交付修改。

本文件是维护规范和补丁台账的唯一索引。Cupertino 的 `docs/upstream-maintenance.md` 只保留入口和消费状态，不复制补丁台账。Git 保存代码版本，GitHub 保存讨论和 review；台账保存两者的对应关系、关键决定和验证入口。不复制全部聊天记录，不另建 YAML 台账或发布机器人。

## 分支和不可变快照

以下约定区分 Alloy 集成仓库和 Zed 贡献 fork。已有 PR 分支保留原名称，不为命名统一而重建 PR。Alloy 的目标默认分支为 `downstream/gpui`；README 和本台账随该分支维护，不提交到纯上游镜像 `main`。

| 引用 | 用途 | 更新规则 |
| --- | --- | --- |
| `upstream/main` | 本地跟踪 Zed 的远端引用 | 执行 fetch 后更新；记录本次使用的完整 SHA。 |
| Alloy 的 `main` | 上游镜像 | 只允许快进同步 Zed，不加入自有提交。出现分叉时先查明原因。 |
| Alloy 的 `downstream/gpui` | 自有 GPUI 的集成分支 | 保留已发布历史；通过普通提交和 merge 更新，不 rebase 已被消费的历史。 |
| `AprilNEA/zed` 的 `fix/<topic>`、`feat/<topic>`、`test/<topic>` | 一个上游 PR 的主题分支 | 从记录的上游基线创建，只携带该主题和明确批准的前置依赖。 |
| `local/<topic>` | 仅自用补丁的工作分支 | 可从集成分支创建；不得直接用来向 Zed 提 PR。 |
| `chore/gpui-sync-YYYYMMDD` | 一次上游同步的候选分支 | 从当前集成分支创建；验收通过后再合入集成分支。 |
| `gpui-alloy/YYYYMMDD.N` | 通过验收的集成快照 tag | 必须为附注 tag；发布后不得移动或删除。它不是 Cargo 包版本号。 |
| `archive/gpui-NNN/YYYYMMDD-N` | topic 改写前的历史 tag | 在 rebase、替换或删除 topic 前保留旧 head；不得移动或删除。 |

上游 PR 分支不得从 `downstream/gpui` 创建。集成分支可能包含材料算法、未提交功能和分发调整，这些改动不能进入无关 PR。

使用独立 worktree 处理 topic 和同步候选。不要在含有未提交组件改动的工作区切换维护分支。不要为了整理历史而混入其他人的暂存内容。

在改写已推送的 topic 前，先确认没有未同步的协作提交。创建并推送旧 head 的历史 tag，确认 tag 可获取后再改写。推送改写结果时使用明确的旧远端 SHA 作为 lease；不要使用无条件 force push。台账必须记录旧 head、新 head、基线和改写原因。

记录 SHA 不能替代保存 Git 对象。每个消费快照和被改写的 PR 版本都必须有长期可获取的 tag。不要只依赖本地 reflog 或 GitHub 暂时保留的旧对象。

## 补丁身份和台账字段

每个独立问题分配一个稳定编号，例如 `GPUI-001`。编号在 rebase、拆分提交或更换 PR 后保持可追溯。拆分出独立问题时分配新编号，并记录原编号关系。

一个主题可以包含实现、文档、回归测试和必要的测试访问器。不要为文件移动、访问器或同一根因的键盘与鼠标修复额外拆 PR。不要把不相关的修改压成一个补丁。

在下游主题提交中使用 `GPUI-Patch: GPUI-001` 这样的 Git trailer。沿用各仓库的提交消息习惯；向 Zed 提交的消息不必携带内部管理字段。

每个补丁记录必须包含下列字段。使用完整 SHA；不存在的记录填写「待建立」或「未执行」，不要推测。

| 字段 | 必须记录的内容 |
| --- | --- |
| 身份 | 编号、单一目的、负责人、影响的模块和公共 API。 |
| 来源 | 上游基线、topic 仓库与分支、topic head、Alloy 集成提交。分别记录 Zed、贡献 fork、Alloy 和 Cupertino 的来源。 |
| 依赖 | 前置补丁编号及原因；区分功能依赖和测试辅助依赖。 |
| 上游 | Discussion、Issue、PR 链接，当前状态和最后核对日期。仅自用时记录原因。 |
| 验证 | 受测提交、命令或操作、环境、结果、证据位置和未验证范围。 |
| 生命周期 | 本地是否集成、上游是否合并、首次包含上游修复的基线、移除或保留原因。 |
| 历史 | 重要决定、PR 提交、基线更新、冲突处理、review 响应、快照采用和补丁退出。 |

分别记录「上游状态」和「本地集成状态」。上游状态使用「未讨论、讨论中、待提交、开放、已合并、关闭未合并、仅自用」。本地状态使用「未集成、已集成未验收、已验收、已由上游替代、已移除」。PR 合并不自动改变本地集成状态。

更新当前状态时保留历史事件。纠正错误记录时写明原记录和纠正原因。每次普通代码提交无需重复整份记录，Git 已保存的内容只需链接。

## 从本地修改形成上游 PR

1. 先确认修改不能在 Cupertino 上层通过现有 API 实现。需要修改 GPUI 时，登记补丁及最小验收行为。
2. 查看 Zed 当前实现、已有 PR 和讨论。确认问题仍存在，不把旧基线的行为当成当前上游状态。
3. 从明确的上游 SHA 创建 topic。移植目标修改，不复制整个 `vendor/` 差异。
4. 将回归测试放进上游可运行的位置。测试不得依赖 Cupertino 的主题、组件或材料模块。
5. 核对目标分支与 topic 的完整 diff。排除 vendor 裁剪、包改名、无关格式变化和仅自用补丁。
6. 在 topic 的确切 head 上完成检查。提交前记录已验证范围及未验证范围。
7. 由作者理解并复核修改后提交 PR。记录 PR URL 和实际 head，不把草稿或等待 CI 的版本标为已验收。

Zed 当前要求一个 PR 只处理一个主题，同时最多开放 3 个 PR，并建议先推进一个 PR。新功能应先取得维护者认可。发起下一项工作前重新核对这些要求，不把本地规范当成上游的接受承诺。[Zed 贡献要求](https://github.com/zed-industries/zed/blob/main/CONTRIBUTING.md)

按当前依赖安排顺序：accessibility 测试支持先于使用该支持的验证；`inert` 先于当前版本的 focus autoscroll；有序 backdrop 先于当前版本的 continuous quad。没有运行时依赖的小修复保持独立。

存在未合并前置 PR 时，优先等待前置 PR 合并。只有维护者接受堆叠 PR 时才采用堆叠，并记录 base 分支及之后的重新定位步骤。

处理 review 时，逐项记录反馈、对应提交和验证结果。作者回复、维护者表情回应、讨论已解决和正式批准是不同状态。CI 的 `action_required`、未执行、失败和通过也必须分别记录。

## 自有版本的集成

通过 cherry-pick 或 merge 将已审查的 topic 纳入集成候选。记录选用方式及 topic 到下游提交的映射。移植提交时保留作者归属；同一补丁的两个 SHA 不代表两个功能。

上游审查可以与自用集成并行。确有需求时，可以在上游合并前消费通过本地验收的补丁，但必须记录未获上游接受的事实。上游反馈导致实现变化后，同步修改下游副本并重新验证。

仅自用补丁必须具有与上游候选相同的提交、测试和记录要求。标明平台限制、性能成本、保留理由以及何时重新评估。不要把「仅自用」作为隐藏散改的理由。

每次可消费版本必须区分三个提交：

| 标记 | 含义 | 记录位置 |
| --- | --- | --- |
| `U` | Zed 上游基线完整 SHA | 补丁记录及快照记录。 |
| `D` | Alloy 的已验证集成提交完整 SHA | 快照 tag；Cupertino 的导入记录。 |
| `V` | Cupertino 导入该快照的提交完整 SHA | 后续台账事件或消费方发布记录。 |

`D` 必须包含本次登记的全部自有修改。`U` 不得冒充 `D`。包含导入记录的提交无法预先填写自身 `V`；从 Git 历史取得 `V`，再写入后续事件或发布记录。

## 上游同步

每周检查一次上游相关变更；检查不等于必须升级。遇到影响当前功能的上游修复、所需新 API 或发布准备时，提前评估同步。不要因为上游有新提交就自动替换消费快照。

1. 选定新的 `U`，记录选择原因及相关上游变更。
2. 从当前集成分支建立同步候选，并记录上一可用 `D` 和 tag。
3. 将新的上游基线合入候选。逐个补丁检查冲突和语义变化，不能把无文本冲突当成兼容。
4. 核对上游已吸收的补丁。记录等效上游实现和 merge commit，停止重复施加补丁。只删除确实重复的实现，不因 PR 合并就 revert 原提交，以免撤销上游已包含的修复。
5. 在 Alloy 的完整 Zed workspace 中运行受影响检查。在隔离的 Cupertino 工作区临时导出候选，再运行消费方验收。失败时保留候选和调查记录，正式消费方继续使用上一可用快照。
6. 两层验收通过后，快进更新集成分支并建立新快照 tag。tag 必须指向实际受测提交；如果集成产生新的 merge commit，必须先验证新提交。
7. 按下一节规则导入 Cupertino，更新台账和消费方检查结果。

上游采用 squash 或不同实现时，不能仅通过 SHA 相等判断补丁已被吸收。确认新基线具有相同行为，并使回归检查继续通过。

上游关闭 PR 时，由维护人明确选择继续自维护、修改后重提或撤回本地修改。没有理由和检查结果，不得静默删除补丁。

## vendor 导入和依赖身份

从固定 `D` 导出实际需要的 GPUI 依赖闭包；验收阶段使用明确记录的候选 SHA。闭包必须包含所选 target 和 features 下的普通依赖、build 依赖、平台依赖、构建脚本、shader、资源和继承的 workspace 配置。另行注明验证所需的 dev 依赖。

每次导入必须记录完整路径清单和导出步骤。不要只记录「复制 GPUI」或「格式清理」。记录 workspace 提取、manifest 改写、文件排除和格式变化的具体规则。

Cupertino 的 `vendor/README.md` 保留来源和使用限制，并链接本台账。导入提交必须同时包含 vendor 内容、需要的 manifest 和 lockfile 更新、必要的 Cupertino 适配及来源记录。审查 import diff 时说明新增和移除的依赖 crate。

完成首次迁移后，不得只在 Cupertino vendor 中修代码而不回写 Alloy。需要临时诊断时，先标记未集成；提交消费版本前把修改纳入新的 `D`，再重新导入。vendor 中只允许记录过的导出转换，不允许未登记的第二份实现。

保留导入代码的许可证、版权归属和适用的 NOTICE 内容。分发前确保每个修改过的 Apache-2.0 文件携带显著的修改声明。新增依赖 crate 时核对实际许可证，不把整个 Zed 仓库视为同一种许可。[Apache-2.0 第 4 节](https://www.apache.org/licenses/LICENSE-2.0)

Cupertino 当前消费方式是 path 依赖 `gpui`，并非 `gpui-pre`。后续改变分发方式时，必须同时处理这些条件：

- Git 消费使用完整 `rev`，不要让生产消费方跟随可移动分支。直接依赖的 GPUI、platform、macros 等配套 crate 必须来自同一个快照。
- Cargo 的依赖别名不改变包身份。将 `gpui-pre` 命名为 `gpui`，不意味着其 `App`、`Window` 可以和另一份 `gpui` 混用。
- 使用 `[patch]` 时，在消费方 workspace 根配置覆盖，并满足被覆盖包的版本要求。依赖库中的 `[patch]` 不会自动传给下游。
- 每次改变来源后检查解析结果，排除意外混入多套 registry、Git 或 path GPUI。保留命令输出或结果摘要。
- Cupertino 当前保持 `publish = false`。`path` 加 `version` 发布后使用 registry 依赖，不会携带这份 vendor 补丁。公开发布前另行确定包名、版本策略和完整依赖分发方案。

依据：[Cargo 依赖来源](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#multiple-locations)和 [Cargo 覆盖规则](https://doc.rust-lang.org/cargo/reference/overriding-dependencies.html#the-patch-section)。

## 验证和证据

检查结果必须绑定确切提交和执行环境。记录 Rust 和 Cargo 的实际版本、target、features 和对应 lockfile；只写「stable」不能确定工具链版本。topic 改写、上游基线变化或冲突处理后，重新执行受影响检查，不复用旧 head 的通过状态。

| 层级 | 必须验证的内容 |
| --- | --- |
| Alloy 的完整 Zed workspace | 上游要求的格式、Clippy、受影响 GPUI crate 的回归测试，以及要求的本机行为检查。 |
| Cupertino | 执行现有 `devenv test`，验证消费方编译、组件行为和相关材料回归。 |
| 原生窗口或渲染 | 验证受影响的真实窗口、输入、焦点、无障碍或 GPU 行为，记录实际支持范围。 |

当前 Cupertino 根 workspace 排除了 `vendor`，所以根目录的 `cargo test --workspace` 不执行 GPUI 自身单元测试。必须在 Alloy 的完整 Zed workspace 中补齐相应检查。不要假定当前裁剪后的 vendor workspace 可以直接独立运行全部上游检查。

一个补丁至少保留一个能揭示问题的检查。对于 bug 修复，记录修改前失败、修改后通过的行为。纯类型声明和直接转发不增加重复测试。跨模块修改按照实际影响补充检查，不为数量增加低价值测试。

原生验证记录 macOS、架构、GPU、Xcode、显示缩放和与结果有关的显示配置。模拟 accessibility 测试不等于 VoiceOver 验收，headless 渲染不等于窗口合成验收。没有多显示器或混合 DPI 的实测时，保留「未验证」。

把构建命令、退出结果和简要结论写入台账。将必要的截图、基准数据和完整日志保存在可长期访问的位置，并记录链接；临时目录或有保留期限的 CI artifact 不能作为唯一证据。不要记录凭据、个人数据或无关完整环境变量。

缺少工具或平台前提时停止对应验收，并记录需要补齐的前提。沙箱导致命令失败时申请提升权限重试，不改命令绕过限制。真实测试失败时修复原因；不削弱断言，不将失败改写为通过。

文档维护只检查来源、链接、状态一致性和格式；文档修改不构成代码验收。提交仍须遵守所在仓库现有的必需检查，不通过修改这份规范来绕过检查。

## 快照和回退记录

每个快照记录 `U`、`D`、tag、补丁编号及版本、检查结果、平台限制和上一可用快照。Cupertino 导入记录另外保存导出规则和 `V`。快照通过验证后才能标记为可消费；不能把创建 tag 当成验证通过。

发生回归时，将 vendor、manifest、lockfile 和必要的消费方适配一起恢复到上一已验证组合。使用新的回退提交保留历史，不移动旧 tag，不只降低版本号。记录触发问题、回退目标、受影响补丁和后续处理。

每次发布前检查以下不变量：

1. 台账能够解释 `U` 到 `D` 的全部行为变化。
2. `D` 和历史 topic 版本仍可通过保存的引用获取。
3. 按记录的导出规则重新生成 vendor，结果与对应 `V` 一致。
4. 依赖解析指向预期的同一套 GPUI。
5. 受测提交等于快照提交，失败和未验证范围没有被隐藏。
6. 上游已吸收、仍需自维护和已移除的补丁状态准确。

## 当前台账

核对日期为 2026-10-03。以下是建立规范时的状态快照，不是实时状态页面。建立原规范时，Cupertino 的 `devenv test` 通过，覆盖当前 Cupertino 工作区的格式、Clippy 和测试。该工作区含未提交修改，不能把结果记为尚未建立的 Alloy 快照或 PR topic 已通过验收；Alloy 的代码检查和新的原生验收尚未执行。

### 当前来源

- 初始上游 `U` 为 `4c841aaf1c4fa613e89a5d77096523d0ff593b56`，与 Cupertino 的 `vendor/README.md` 一致。这是用于迁移的固定基线，不是上游最新提交。
- 核对时的 Cupertino `HEAD` 为 `541d05454899085fcd8fc3b188344dc63002f8c3`，分支为 `feat/component-library`，并含尚未提交的组件和 vendor 改动。
- 本地 `main` 和 `downstream/gpui` 已从上述 `U` 建立。初始化仅加入维护文档；GPUI 源码保持该基线。完整下游快照 `D` 和可消费快照 tag 尚未建立。
- 现有 vendor 首次导入包含在 Cupertino 提交 `9c8b4e23c299006e2440eada4043dc1d04c2a750`；之后存在多次修改，不能把该提交当成当前完整快照。
- 当前导出转换及逐文件清单尚未形成可重复执行的完整记录。首次迁移时必须补齐，不能宣称当前 vendor 已由某个 `D` 重现。

### 已有上游 PR

这两项尚未进入当前 vendor。保留现有 PR，不能为同一主题再开一个 PR。

| 编号 | 主题与分支 | 上游状态 | 下一步 |
| --- | --- | --- | --- |
| GPUI-010 | 原生锚定弹窗；`feat/macos-anchored-popup` | [#64353](https://github.com/zed-industries/zed/pull/64353) 开放；与 `main` 冲突；没有人工 review；CI 为 `action_required`。 | 保存旧 head 后解决冲突，重新执行相关检查。原生多屏和混合 DPI 仍未验证。 |
| GPUI-011 | 无标题 popup 去除原生装饰；`fix/macos-popup-decorations` | [#64430](https://github.com/zed-industries/zed/pull/64430) 开放；无合并冲突；已补截图，但仍为 `CHANGES_REQUESTED`；CI 为 `action_required`。 | 核实 borderless 拖动风险，再推动复审和 CI。机器审查提示尚未独立复现。 |

GPUI-010 当前 head 为 `619d4b2b3da3040e61c027a394aa5417123e92d0`。GPUI-011 当前 head 为 `22df6ff9402b46943d5948e767b4c7ad493d69c3`。两个分支都位于 `AprilNEA/zed`。当前 head 已由下文的本地归档 tag 保存；原始创建基线和更早的改写记录仍需补齐；PR 当前 base 的 SHA 不能代替原始创建基线。

两项 CLA 均已通过。测试 workflow 均没有启动测试 job，不应记为代码失败或 CI 通过。依据：[锚定弹窗检查](https://github.com/zed-industries/zed/actions/runs/35315963900)、[装饰修复检查](https://github.com/zed-industries/zed/actions/runs/35315318419)、[补图要求](https://github.com/zed-industries/zed/pull/64430#pullrequestreview-5249581274)和[补图回复](https://github.com/zed-industries/zed/pull/64430#issuecomment-5732521089)。

### 当前本地补丁

以下 9 项已存在于 Cupertino 当前 vendor，其中 8 项是上游候选，1 项建议仅自用。9 项在 Alloy 中均为「未集成」，上游候选尚未建立对应 PR topic。表中的提交均属于 Cupertino 历史，不是 Alloy 中独立的补丁提交，也不是已经建立的 `D`。检查路径相对于 Cupertino 仓库；尚未提交的文件没有可固定的远端链接，迁移时补记来源提交和验证证据。

| 编号 | 主题 | 当前来源与检查入口 | 上游安排 |
| --- | --- | --- | --- |
| GPUI-001 | handler 移除后取消键盘和鼠标旧按压 | 键盘修复在 `ef5d17f00df73cab140b071ff23ee220738b6aff`；鼠标修复尚未提交。检查 `crates/gpui-cupertino/tests/button.rs` 和 `crates/gpui-cupertino/tests/choice_controls.rs`。 | 一个修复 PR；需提取纯 GPUI 回归测试。 |
| GPUI-002 | interceptor 阻止后续 raw capture | 尚未提交；检查 `crates/gpui-cupertino/src/components/tooltip/tests.rs`。 | 一个独立 PR；先明确停止传播的行为契约。 |
| GPUI-003 | TestWindow accessibility 支持 | `1eb328bca34ca7c51426e9472110d04111aee385`；检查 `crates/gpui-cupertino/tests/accessibility.rs`。 | 一个测试支持 PR，可先于 GPUI-005 的 accessibility 验证。 |
| GPUI-004 | Spring 使用 scheduler clock | 修改包含于 `ebba53ae60327ac0050d8f0be543642d90996dbb`，待提取；检查 `crates/gpui-cupertino/tests/motion.rs`。 | 一个独立小修复，不重复提交上游已有的 Spring API。 |
| GPUI-005 | `gpui::inert` | 尚未提交；检查 `crates/gpui-cupertino/src/components/panels_tests/inert.rs`。 | 一个新 API PR，先讨论；必须包含 deferred 和缓存行为。 |
| GPUI-006 | `Div::autoscroll_on_focus` | 尚未提交；检查 `crates/gpui-cupertino/tests/scroll_focus.rs`。 | 一个新 API PR；当前实现依赖 GPUI-005。 |
| GPUI-007 | 有序 backdrop 与 Metal 实现 | 实现在首次导入及后续材料提交中，待提取；检查 `crates/gpui-cupertino/tests/scene.rs` 和 `docs/validation.md`。 | 一个完整渲染主题，先讨论平台契约和性能成本。 |
| GPUI-008 | Continuous rounded quad | 修改包含于 `be245ff8b0d15db9e9b4e90b64ffd90fcd2788b6`，待提取；检查 `crates/gpui-cupertino/tests/continuous_corners.rs`。 | 一个独立绘制能力 PR；当前实现依赖 GPUI-007。 |
| GPUI-009 | Inactive Clear 与 ColorSync | 修改包含于 `be245ff8b0d15db9e9b4e90b64ffd90fcd2788b6`；检查 `docs/validation.md`。 | 建议仅自用；依赖 GPUI-007 和 GPUI-008；记录 Apple GPU、SDR 和色彩配置限制。 |

当前维护范围共 11 个主题，不代表需要再开 11 个 PR。已有 2 个开放 PR，另有 8 个上游候选和 1 个仅自用候选。测试访问器跟随所属补丁，不另算主题。

### 后续记录模板

补丁进入正式维护时，在本节后添加记录。初期维护同一份台账，不额外建立目录或生成器。

```text
补丁编号与目的：
负责人：
上游基线 U：
topic 仓库、分支与完整 head：
改写前保存的历史 tag：
下游实现提交及依赖编号：
Discussion / Issue / PR：
上游状态与核对日期：
本地集成状态与采用快照：
验证提交、环境、命令或操作、结果、证据：
未验证范围：
被上游替代或继续自维护的条件：
历史事件：日期、决定、旧引用、新引用、原因、验证结果。
```

```text
快照 tag 与日期：
上游 U：
Alloy URL 与下游 D：
补丁编号及对应实现版本：
上一可用快照：
Alloy 检查结果与证据：
Cupertino 检查结果与证据：
平台限制和未验证范围：
vendor 路径清单与导出转换规则：
Cupertino 导入提交 V：由导入后的历史事件补充。
本次新增、替代、保留和移除的补丁：
```

## 从当前状态迁移

1. 保留现有未提交工作，由各项工作的负责人完成验收和提交。不要把当前工作区直接标成可消费快照。
2. 按下文步骤推送 Alloy 分支和两个 PR 的归档 tag，确认远端可获取后再改写 topic。补记两个 PR 的原始创建基线。
3. 从当前 vendor 提取 GPUI-001 至 GPUI-009，形成独立主题提交。保留已有作者归属和测试证据。
4. 在 Alloy 中形成候选提交并验证组合，记录候选 SHA。
5. 记录依赖闭包和导出转换，在隔离的消费方工作区从候选重建 vendor。解释所有差异，再运行 Cupertino 检查。
6. 两层验收通过后，将受测提交确认为首个 `D`，建立快照 tag，再提交正式导入。记录完整的 `U`、`D`、`V` 对应关系。此后停止维护未回写 Alloy 的 vendor 补丁。
7. 优先推进已有 PR。新增 PR 时遵守上游并发上限和功能讨论要求。

只有完成这些步骤，才可以称为「已建立可重现的自维护 GPUI 版本」。仅保存分支名称和 PR 链接不满足这一条件。

## 仓库初始化记录

2026-10-03：创建公开仓库 `AprilNEA/gpui-alloy`。保留完整 Zed 历史；本地 `main` 指向初始 `U`，`downstream/gpui` 承载 Alloy 文档。原有 `AprilNEA/zed` 仓库和 PR 分支未被改写。

| 本地 remote | 地址 | 用途 |
| --- | --- | --- |
| `origin` | `https://github.com/AprilNEA/gpui-alloy.git` | Alloy 分支和快照。 |
| `upstream` | `https://github.com/zed-industries/zed.git` | 获取 Zed 上游。 |
| `pr` | `https://github.com/AprilNEA/zed.git` | 获取或推送上游 PR topic。 |

默认推送远端配置为 `origin`。向贡献 fork 推送 topic 时必须显式指定 `pr`。初始化提交和本地附注 tag 按维护人要求暂不签名；使用命令级配置，不修改全局签名设置。

| 本地归档 tag | 对应 PR | 保存的完整 head |
| --- | --- | --- |
| `archive/gpui-010/20261003-1` | #64353 | `619d4b2b3da3040e61c027a394aa5417123e92d0` |
| `archive/gpui-011/20261003-1` | #64430 | `22df6ff9402b46943d5948e767b4c7ad493d69c3` |

初始化时这些分支、文档和 tag 仅在本地，尚未推送。GitHub 默认分支仍为 `main`；首次推送后由维护人改为 `downstream/gpui`。本地归档不满足远端持久化要求，确认推送成功前不得改写已有 PR 分支。

由维护人在 Alloy 仓库运行以下命令。显式列出引用，避免把上游的其他 tag 或分支一并推送。

```sh
git push --atomic origin main downstream/gpui refs/tags/archive/gpui-010/20261003-1 refs/tags/archive/gpui-011/20261003-1
gh repo edit AprilNEA/gpui-alloy --default-branch downstream/gpui
git fetch origin
git branch --set-upstream-to=origin/downstream/gpui downstream/gpui
git ls-remote origin refs/heads/main refs/heads/downstream/gpui 'refs/tags/archive/gpui-*'
```

首次推送后，追加日期和远端核对结果。保留本次初始化记录。

GitHub Actions 已在仓库设置中关闭。上游 workflow 包含 Zed 的发布和服务任务；建立 Alloy 专用 CI 并核对触发条件后，再决定启用哪些 workflow。关闭 Actions 不免除本地验收要求。

本次初始化只验收仓库可见性、Git 历史连通性、分支和归档目标、源码与 `U` 的一致性以及维护文档。没有执行 Alloy 的 GPUI 编译、单元测试或原生验收；没有发布 crate、可消费 tag 或新的上游 PR。
