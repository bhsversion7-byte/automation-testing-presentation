# 从 Testing Basics 到 Serenity BDD：银行后台 Developer 知识库

> **Audience**：NPA Team Developer、QA、Team Lead。\
> **Tech stack**：Java + Maven + Cucumber + Serenity BDD；NPA 项目关注网站 UI、角色权限、API 和阶段流转。\
> **Last source check**：2026-10-12；文档版本：1.3。\
> **Scope**：测试基础、自动化价值、回归策略、BDD 协作、Serenity 实施与报告。\
> **Example policy**：支付规则、接口、状态码和数据都是虚构教学契约，不代表 DBS 的实际系统、业务要求或内部政策。本文没有访问内部项目，也没有执行银行系统测试。

**项目背景补充（2026-10-12）**：NPA Team 的网站管理 financial proposal 的 Pre-NPA、In Progress、In Live 及 Expired 区域，Pre-NPA 有 Supporting Units、相关 Head、COO 等角色的 sign-off。当前项目学习重点是网站操作、角色权限、审批记录保存和阶段流转。本文的基础概念仍适用；支付 API 代码保留为独立教学例子，NPA sign-off 示例请看 [展示 PPT](../output/Serenity_Testing_Reports_Compatible.pptx) 和 [对应讲稿](../output/Serenity_Testing_Reports_Speaker_Notes.md)。具体审批和阶段条件尚未确认。

## 阅读导航

| 你的问题 | 推荐章节 |
| --- | --- |
| Automation 对 QA 和项目有什么作用？ | [概念关系](#section-1)、[自动化价值](#section-3)、[测试组合](#section-4) |
| Testing、Regression、BDD 到底是什么关系？ | [Testing Basics](#section-2)、[Regression](#section-5)、[BDD](#section-6) |
| Serenity、Cucumber、JUnit、Maven 各做什么？ | [工具链](#section-7)、[工程配置](#section-8) |
| 怎样写并运行第一个 Serenity BDD API 场景？ | [实践示例](#section-9) |
| 怎样组织回归、接入 CI、看报告和定位失败？ | [Regression Suite / CI](#section-10)、[报告](#section-11)、[排查](#section-12) |
| 怎样向组长展示，之后继续学习？ | [银行后台语境](#section-13)、[学习路线](#section-14)、[展示建议](#section-15)、[逐页讲稿](../output/Serenity_Testing_Reports_Speaker_Notes.md) |

<a id="section-1"></a>
## 1. 先建立整体概念：这些词处于不同维度

最需要记住的是：

**Testing 是评估质量的活动；Regression 是变更后的测试目的；Automation 是用工具执行和支持测试的方式；BDD 是围绕业务行为建立共同理解的开发协作方法；Serenity 是支持自动化验收、回归及报告的工具。**

| 概念 | 回答什么问题？ | 一个后台业务例子 |
| --- | --- | --- |
| Testing / 软件测试 | 如何评估系统是否符合要求，并发现风险？ | 检查超限支付是否被拒绝 |
| Regression Testing / 回归测试 | 改动是否破坏已有行为？ | 修改限额逻辑后，正常支付和冻结账户规则是否仍正确 |
| Automation Testing / 自动化测试 | 哪些测试活动由程序和工具执行？ | 自动准备账户、调用 API、校验结果、生成报告 |
| BDD / Behaviour-Driven Development | 业务、QA、Dev 如何明确系统应该怎样表现？ | 共同讨论“等于剩余额度时是否允许” |
| Cucumber | 如何执行可读的行为规格？ | 把 `.feature` 中的步骤映射到 Java 方法 |
| Serenity BDD | 如何组织自动化并形成可读证据？ | 记录步骤、结果、需求关联，生成 HTML 报告 |

上表是对概念的归纳，依据 [ISTQB CTFL §1、§2.2.3、§6](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)、[Cucumber BDD](https://cucumber.io/docs/bdd/) 和 [Serenity Introduction](https://serenity-bdd.github.io/docs/guide/user_guide_intro)。

因此，一条测试可以同时是：**API test + functional test + automated test + regression test + BDD acceptance scenario**。这些标签可以共同描述同一条用例。

注意：API 描述交互入口，Unit/System 描述测试层级，Functional 描述关注的质量方面。不能把这些词当成互斥选项。

<a id="section-2"></a>
## 2. Testing Basics：Developer 必须掌握的基础

### 2.1 什么是软件测试？

Software Testing 是评估软件及相关工作产物质量、发现缺陷、降低运行失败风险的一组活动。它包括需求和代码的静态检查，也包括运行系统的动态验证。测试结果为 release decision 提供证据，但通过既定用例并不能证明系统不存在缺陷。[ISTQB CTFL §1.1–1.3](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)

可以从两种关注点理解：

- **Verification**：实现是否符合规格，例如限额比较逻辑是否符合已确认规则。
- **Validation**：交付是否满足用户需要，例如后台操作人员是否能正确处理被拒绝的支付。

Testing 的结果通常是发现异常并收集证据；**Debugging** 则是定位原因并修复。自动化报告可以帮助 debugging，但不会自动替团队判断正确的业务规则。

### 2.2 QA、Testing、Developer 的关系

正式术语中，**QA（Quality Assurance）偏向过程改进和缺陷预防**，Testing 是偏向产品的质量控制活动。工作中大家也常用“QA”称呼测试同事或测试岗位，要根据语境理解。[ISTQB CTFL §1.2.2](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)

下面是建议的协作分工；具体 owner 以团队安排为准。

| 角色 | 主要贡献 | 共同参与 |
| --- | --- | --- |
| Business / BA / Product | 业务规则、用户目标、验收标准 | 澄清例子与边界 |
| QA / Test Engineer | 测试设计、风险分析、探索性测试、失败证据 | 选择 regression scope，评审自动化 |
| Developer | 可测试设计、unit/integration tests、实现与修复 | 编写维护自动化、调查失败 |
| Team Lead / Release Owner | 优先级、资源和交付决策 | 理解测试覆盖与剩余风险 |
| Platform / DevOps | 环境、流水线、构建及产物保存 | 支持可重复执行 |

质量由整个团队共同负责。Developer 不应把“QA 会测”当作跳过自身验证的理由；QA 的价值也不能用写了多少脚本衡量。

### 2.3 一条完整 Test Case 包含什么？

| 元素 | 含义 | 教学示例 |
| --- | --- | --- |
| Test basis | 设计测试的依据 | “金额不得超过当日剩余额度”的已确认需求 |
| Test condition | 需要验证的行为或条件 | 支付金额刚好等于剩余额度 |
| Preconditions | 前置条件 | 账户 active、币种 SGD、余额充足 |
| Test data | 输入和已有状态 | 剩余额度 SGD 1,000.00；金额 SGD 1,000.00 |
| Test steps | 执行动作 | 调用支付校验接口 |
| Expected result | 独立确定的预期 | 返回允许校验结果 |
| Actual result | 系统实际表现 | 响应里的 decision |
| Assertion / Test oracle | 判定依据与比较 | `decision == ALLOWED`，依据已确认规则 |
| Cleanup | 恢复或隔离数据 | 清理独立测试记录，或使用只读、固定 fixture |
| Traceability | 关联需求与问题 | 对应 story、业务规则、缺陷编号 |

**执行成功不等于验证成功。** HTTP 请求没有抛异常，只说明请求完成；测试还必须验证业务结果。HTTP 200 也可能携带错误的业务状态。

Test oracle 是判断正确与否的依据。不要直接复用被测代码计算期望值，否则同一个错误可能同时存在于实现和测试里。

### 2.4 常见 Test Levels、Test Types 与团队用语

| 术语 | 关注点 | 后台例子 |
| --- | --- | --- |
| Component / Unit testing | 单个组件内部行为 | `amount <= remainingLimit` 的边界判断 |
| Component integration testing | 组件之间交互 | 校验服务与额度 repository 的接口 |
| System testing | 整个系统满足要求 | 支付服务暴露 API 的整体行为 |
| System integration testing | 系统之间交互 | 支付服务、账务和消息系统的协作 |
| Acceptance testing / UAT | 是否满足业务验收目标 | 业务代表确认操作流程和业务规则 |
| Functional testing | 系统做了什么 | 允许、拒绝、撤销、重复请求处理 |
| Non-functional testing | 系统表现得怎样 | 性能、可靠性、安全性、可用性 |
| Contract testing | 交互双方对接口的假设是否一致 | schema、字段语义、错误格式和兼容性 |
| End-to-End / E2E | 覆盖完整业务路径 | 发起→审批→处理→账务→查询 |

前五行对应 ISTQB 的测试层级；后面是测试关注点或实践术语。API test 可以在多个层级发生。SIT 在不同组织里也可能指不同范围，需要确认团队定义。[ISTQB CTFL §2.2](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)

### 2.5 Static vs Dynamic、Manual vs Automated

这也是不同维度：

- **Static testing**：不运行被测软件，例如需求评审、code review、静态分析。
- **Dynamic testing**：运行软件并检查结果，例如执行 unit test 或 API test。
- **Manual testing**：人执行或探索，适合未知问题、体验判断和临时调查。
- **Automated testing**：工具执行已定义的检查及支持活动，适合反复验证。

自动化可以支持静态和动态活动。Manual 与 Automated 可以共同构成 Regression Suite。

### 2.6 怎样设计有价值的测试？

下面是对本教学规则的测试设计示范：**金额必须为正，且不超过当日剩余额度**。这些是本文假设，真实规则应先确认。

| 方法 | 如何思考 | 示例 |
| --- | --- | --- |
| Equivalence Partitioning / 等价类 | 把输入分成预计行为相似的集合 | 合法金额、非正金额、超限金额 |
| Boundary Value Analysis / 边界值 | 检查规则切换的位置 | 999.99、1,000.00、1,000.01；0.00、0.01 |
| Decision Table / 决策表 | 多个条件组合会怎样？ | 账户状态 × 余额是否足够 × 额度是否足够 |
| State Transition / 状态迁移 | 当前状态允许什么动作？ | pending approval 能否进入 approved；settled 能否再次处理 |
| Exploratory testing / 探索性测试 | 用业务知识探索未预先穷举的情况 | 超时、重复提交、异常恢复、操作误解 |

如果金额使用最小货币单位，SGD 1,000.00 = 100,000 cents。金额精度与 rounding 规则要显式定义，Java 通常使用整数 minor units 或 `BigDecimal`，避免用二进制浮点数直接判断金额相等。

方法名称可参考 [ISTQB CTFL §4](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)；表内支付数据和实现建议是教学设计。

### 2.7 七个测试原则：用于约束预期

1. 测试可以揭示缺陷存在，不能证明没有缺陷。
2. 除极简单情况外，穷尽所有组合通常不可行。
3. 尽早测试和评审，可以更早发现误解。
4. 缺陷可能集中在少数组件，历史缺陷可帮助确定重点。
5. 长期只重复相同测试会降低发现新问题的能力，需要更新用例。
6. 测试策略取决于系统、风险和使用背景。
7. 即使已发现的缺陷被修复，如果系统不满足用户需要，产品仍可能失败。

以上为 [ISTQB CTFL §1.3](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf) 的概括；不是要求每个项目采用同一套固定测试数量。

<a id="section-3"></a>
## 3. Automation Testing：对 QA 与项目意味着什么？

### 3.1 自动化不是只让程序点按钮

在本文语境里，Test Automation 是用程序和工具支持测试：准备输入和环境、执行动作、比较预期与实际、记录证据、生成反馈。例如：

```text
Prepare known test data
        ↓
Call the API / interact with the system
        ↓
Assert business outcomes and relevant side effects
        ↓
Record evidence and report the result
```

这条流程中，最核心的是 **Assertion**。没有正确预期和断言，脚本可能只是在自动执行一个业务流程。

### 3.2 对 QA 的价值

- 重复检查可以稳定地反复执行，减少每次 release 的机械劳动。
- 更快得到变更反馈，QA 可以将时间用于测试设计和探索。
- 场景、断言和报告可供评审，帮助解释失败和覆盖情况。
- QA 与 Developer 共同维护测试资产，而不是每次手工重做全部检查。

对 QA 的要求也随之增加：需要懂业务风险、数据设计、断言质量、失败分类，以及脚本维护。自动化并不会取代 QA 的判断。

### 3.3 对项目的价值与成本

| 方面 | 可能的收益 | 必须投入或关注的成本 |
| --- | --- | --- |
| Feedback speed | 每次变更都能快速得到已覆盖行为的结果 | 构建、执行和环境时间 |
| Repeatability | 一致执行步骤与断言 | 数据、依赖和配置必须可控 |
| Release confidence | 持续验证重要业务规则 | 覆盖遗漏仍然存在 |
| Maintainability | 重构后有保护网 | 用例与规则要一起演进 |
| Evidence | 留存结果与上下文 | 产物保存、权限与数据处理 |
| Team efficiency | 减少重复检查 | 开发、维护、失败调查、基础设施 |

这些收益和风险与 [ISTQB CTFL §6.2](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf) 一致；具体项目是否节省成本，必须由自身数据验证。

一个用于讨论的成本模型：

```text
Net benefit ≈ avoided repetitive manual effort
              − test implementation effort
              − maintenance effort
              − failure triage effort
              − infrastructure cost
```

这不是财务公式，也不承诺“自动化一定节省 X%”。高频、稳定、风险高的检查通常更值得先投入。

### 3.4 优先自动化什么？

| 适合优先投入 | 需要谨慎评估或保留人工判断 |
| --- | --- |
| 高频执行的核心 regression | 一次性、很少重跑的检查 |
| 稳定且明确的业务规则 | 规则尚未确认或不断变化的流程 |
| 可准确断言的输入、输出和状态 | 主观体验、视觉审美、未知风险探索 |
| 金额边界、权限拒绝、重复请求等风险 | 难以控制的数据和第三方环境 |
| 可独立准备与清理的数据 | 依赖其他场景先执行的长链条 |

这是本文面向后台项目的工程建议。可以按 **业务风险、执行频率、规则稳定性、数据可控性、维护成本** 排序，而不是按“哪个最容易写”排序。

<a id="section-4"></a>
## 4. 自动化测试组合：为什么后台项目应重视低层级检查？

Test Pyramid 是组织测试组合的经验模型：大量快速且局部的测试，加上一些集成检查和少量完整 E2E 测试。它不规定固定百分比，应根据项目风险调整。[The Practical Test Pyramid](https://www.martinfowler.com/articles/practical-test-pyramid.html)

```text
           /  Selected E2E  \        完整路径，环境依赖多
          /------------------\
         / API / Integration  \     验证契约、状态和系统交互
        /----------------------\
       / Fast Unit / Component  \   规则边界、计算、状态逻辑
      /__________________________\
```

对本文支付规则，可以这样分配：

- **Unit**：详细覆盖金额比较、精度、边界和异常分支。
- **API/Integration**：确认外部请求映射正确、业务错误正确、依赖交互符合预期。
- **E2E**：选择关键业务路径验证系统协同，必要时包含 UI。

不需要把每个金额组合都通过完整 UI 或长链 E2E 执行。较低层级的测试更容易定位问题；较高层级的测试用于发现低层级看不到的集成风险。

**API-first 是本学习案例的选择。** 它适合后台规则与接口学习，但如果需求涉及 UI 权限、操作体验或前后端集成，仍需要相应的 UI 检查。

<a id="section-5"></a>
## 5. Regression Testing：变更之后，原有行为是否仍然正确？

### 5.1 定义与触发条件

Regression Testing 用来检查变更是否造成不良影响。这些影响可能发生在改动模块、其他模块、连接的系统或环境。触发因素可以是新功能、bug fix、refactoring、配置变动、依赖升级、数据库或环境迁移。[ISTQB CTFL §2.2.3、§2.3](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)

它不只是“测旧功能”，更具体地说，是**根据变更风险重新验证应该保持的行为**。同一条验收用例，在功能首次交付时验证新需求，以后重跑时就可以承担 regression 的作用。

### 5.2 Regression、Confirmation、Smoke 的区别

| 名称 | 核心问题 | 支付示例 |
| --- | --- | --- |
| Confirmation / Retest | 原来那个缺陷是否修好了？ | 原来 1,000.01 被允许，修复后应被拒绝 |
| Regression | 修复或其他变更是否破坏了应该保留的行为？ | 999.99、1,000.00、冻结账户和其他币种仍按规则工作 |
| Smoke | 当前构建是否基本可用，值得继续测？ | 服务可访问、基础认证正常、一个代表性业务路径成功 |
| Full regression | 在约定范围内执行完整回归集合 | 运行项目定义的所有 regression 用例 |

Confirmation 与 Regression 的区别依据 [ISTQB CTFL §2.2.3](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)。Smoke 在这里采用常见的团队实践含义，其范围需要由团队定义。所谓 full regression 也不代表穷尽所有可能行为。

### 5.3 Automation 与 Regression 的关系

| | Manual execution | Automated execution |
| --- | --- | --- |
| Regression purpose | 人工重测受影响的业务流程 | Pipeline 重跑已有 API、unit 或 E2E 用例 |
| Other purposes | 人工探索新需求或验证体验 | 自动验证新功能、执行性能检查等 |

Regression 可以手工执行；Automated Test 也可以验证新功能。回归用例通常频繁重跑，因此是自动化的良好候选。[ISTQB CTFL §2.2.3](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)

### 5.4 从变更到 Regression Scope

以下是建议流程：

1. **Identify the change**：代码、配置、数据模型还是依赖发生了什么变化？
2. **Impact analysis**：调用者、下游、共享模块、状态与数据可能如何受影响？
3. **Choose checks**：直接受影响场景 + 相关业务路径 + 核心 smoke。
4. **Execute at suitable levels**：优先快速局部检查，再执行必要的系统交互检查。
5. **Triage failures**：判断产品、测试、数据、环境或规则变化。
6. **Review remaining risk**：哪些部分没有执行或没有覆盖，是否可接受？

限额改动的示例回归矩阵：

| 业务规则 / 风险 | 代表性例子 | 建议验证方式 | 说明 |
| --- | --- | --- | --- |
| 正常金额被错误拒绝 | 低于额度 | Unit + API | 核心成功行为 |
| 等于额度比较错误 | 刚好等于额度 | Unit + API | 明确 `<=` 规则 |
| 超限被错误允许 | 超额度一个最小单位 | Unit + API | 核心负向场景 |
| 余额不足逻辑受影响 | 额度够、余额不足 | Unit + API | 区分两种拒绝原因 |
| 账户状态规则受影响 | 冻结账户请求支付 | Unit + API | 不能只测 happy path |
| 权限或审批绕过 | 不具备权限的操作请求 | API + 选定 E2E | 以实际身份模型为准 |
| 重试导致重复处理 | 同 idempotency key 重复提交 | Integration + E2E | 适用于会产生业务副作用的接口 |
| 处理失败导致状态不一致 | 超时、回滚、消息重放 | Integration + 选定 E2E | 以真实系统机制为准 |

后五行不是第 9 节代码已覆盖的内容，而是扩展回归的候选。要结合团队实际架构选择。

<a id="section-6"></a>
## 6. BDD：先共同理解行为，再自动验证

### 6.1 BDD 的三个实践

Cucumber 将 BDD 描述为围绕具体例子的协作过程，包括 **Discovery → Formulation → Automation**。[Cucumber BDD](https://cucumber.io/docs/bdd/)

| 阶段 | 做什么 | 产物 |
| --- | --- | --- |
| Discovery | Business、QA、Dev 讨论规则、例子与疑问 | 已确认规则、代表性例子、待回答的问题 |
| Formulation | 用清楚的业务语言记录例子 | 可读的行为规格，如 Gherkin |
| Automation | 将规格连接到系统并验证 | 可执行用例和执行证据 |

常说的 **Three Amigos** 是三种视角：业务意图、测试风险、技术实现，不要求每次刚好三个人。

### 6.2 先讨论什么，再写什么？

User Story 的教学例子：

> 作为支付发起方，我希望系统在金额超过当日剩余额度时拒绝校验，以遵守已配置的限额规则。

在写 `.feature` 前，应讨论：

- “额度”是单笔额度、当日累计额度，还是当日剩余额度？
- 等于额度时能否允许？金额精度是什么？
- 余额不足与超限同时发生，返回哪个错误？
- 额度按账户、客户、角色还是渠道计算？
- 何时重置当日额度？哪个时区定义“当日”？
- 请求重试或并发校验是否需要保留、占用或锁定额度？

这些问题展示 Discovery 的价值。本文示例只选一个已明确的简化规则：**正金额且不超过当日剩余额度；账户 active、币种 SGD、余额充足；校验本身不占用额度。**

可以用 Example Mapping 分别记录 story、rules、examples 和 questions。[Cucumber Example Mapping](https://cucumber.io/docs/bdd/example-mapping/)

### 6.3 Gherkin 与 Given / When / Then

| Keyword | 表达的含义 |
| --- | --- |
| `Feature` | 一组相关业务行为 |
| `Rule` | 要说明的业务规则 |
| `Scenario` | 一个具体行为例子 |
| `Given` | 已知的前置条件与状态 |
| `When` | 触发动作或事件 |
| `Then` | 可观察、可验证的结果 |
| `And` / `But` | 延续前一个步骤的语义 |
| `Background` | 每个相关场景运行前执行的共同前置步骤 |
| `Scenario Outline` + `Examples` | 用多组数据展开相同的行为模式 |
| `@tag` | 场景组织、筛选或元数据 |

关键字语义与 Outline 展开方式见 [Gherkin Reference](https://cucumber.io/docs/gherkin/reference/)。`Given`/`When`/`Then` 便于表达 Arrange/Act/Assert，但并不强制某种底层实现。

```gherkin
Scenario: Reject a payment above the remaining daily limit
  Given an active account with a remaining daily limit of SGD 1000.00
  When a payment of SGD 1000.01 is validated
  Then the validation is rejected because the daily limit is exceeded
```

这一场景表达业务行为。底层可以通过 API 实现，不一定要经过浏览器。

### 6.4 好场景与常见误区

- 一条场景聚焦一个主要行为；名字应能解释为什么需要它。
- 使用业务术语，避免把每个 click、sleep、SQL 都写进规格。
- 前置状态必须清楚，预期结果必须可观察。
- 重要的拒绝行为、边界和状态迁移与成功行为一样需要讨论。
- 不要让 Scenario B 依赖 Scenario A 先运行。
- 不需要把所有 unit tests 翻译成 Gherkin；只有规格和协作价值足够时才值得增加这一层。

BDD、TDD 与 Data-Driven Testing 可以共存：TDD 常通过小范围测试驱动代码设计；BDD 更强调行为与共同语言；Data-Driven Testing 则通过多组数据复用测试逻辑。

**写了 Given/When/Then，并不能单独证明团队完成了 BDD 协作。** 场景应先得到相关角色对业务含义的认可。

<a id="section-7"></a>
## 7. Serenity BDD 的定位：它在工具链里做什么？

Serenity BDD 是 Java 生态中的开源测试库，支持自动化验收与回归，并将测试步骤和结果组织成报告与 Living Documentation。它可以与 Cucumber、REST Assured、Selenium 和 Screenplay 配合。[Serenity Introduction](https://serenity-bdd.github.io/docs/guide/user_guide_intro)

| 工具 / 概念 | 职责 | 本文的选择 |
| --- | --- | --- |
| Gherkin | 行为规格的文本语法 | `.feature` |
| Cucumber | 解析场景、匹配并执行 step definitions | Cucumber JVM |
| Java step definitions | 业务文字与实际测试动作之间的桥梁 | `@Given`、`@When`、`@Then` |
| Serenity BDD | 步骤记录、测试支持、报告与需求呈现 | Serenity Cucumber integration |
| REST Assured / SerenityRest | HTTP 请求、响应校验及 Serenity 记录 | API testing |
| JUnit Platform + Cucumber engine | 测试发现与启动 | `@Suite` |
| Maven | 依赖、编译、测试 lifecycle、报告生成 | `mvn clean verify` |
| CI system | 自动触发、执行、保存产物与反馈 | 依团队现有平台 |

```text
Business rule + agreed examples
            ↓
Feature file (.feature)
            ↓ Cucumber matches the text
Java Step Definitions
            ↓
Reusable API Actions / Screenplay Tasks
            ↓
System under test → observable result → assertion
            ↓ Serenity records execution
HTML report / Living Documentation

Maven + JUnit Platform 启动执行；CI 在选定时机触发 Maven。
```

Serenity 的“需求覆盖”报告与代码覆盖率不同。前者依赖需求结构、场景关联及实际执行结果；代码覆盖率通常另用 JaCoCo 等工具。报告可证明已定义检查的结果，不能自行证明所有业务风险都已覆盖。[Serenity Reports](https://serenity-bdd.github.io/docs/reporting/the_serenity_reports)

### 7.1 Step Libraries、Page Objects、Screenplay 怎样选择？

| 组织方式 | 用途 | 学习建议 |
| --- | --- | --- |
| Step / Action classes | 将 API 调用或业务动作封装成复用方法 | 初学先掌握；第 9 节采用此方式 |
| Page Object | 把 UI 定位和页面交互封装起来 | 团队测试 UI 时再学习 |
| Screenplay | 用 Actor、Ability、Task、Interaction、Question 描述行为 | 项目使用时遵循；不必与入门概念同时全部引入 |

Screenplay 中，Actor 代表参与者，Ability 代表可用能力，Task 表达目标，Interaction 表达底层动作，Question 获取需要断言的状态。[Screenplay Fundamentals](https://serenity-bdd.github.io/docs/screenplay/screenplay_fundamentals)、[Screenplay REST](https://serenity-bdd.github.io/docs/screenplay/screenplay_rest)

BDD 不强制 Screenplay，Serenity 也不强制每个测试使用 Cucumber。技术组织方式应与现有项目保持一致。

<a id="section-8"></a>
## 8. 开始前：项目结构、版本与 Maven 配置

### 8.1 优先理解团队现有工程

拿到项目后，先查这些位置：

| 位置 | 需要回答的问题 |
| --- | --- |
| `pom.xml` | JDK、Serenity、Cucumber、JUnit 版本是什么？哪些 plugins 执行测试和报告？ |
| `src/test/resources/features` | 业务场景怎么分目录、怎样使用 tags？ |
| Test Suite / Runner | 场景路径和 glue package 怎么指定？ |
| Step definitions | 文字怎样映射到 Java？测试状态保存在哪里？ |
| Actions / Tasks / Questions / Pages | API 和业务动作怎么复用？ |
| `serenity.conf`、properties、Maven profiles | 环境、地址、浏览器和报告怎样配置？ |
| Hooks / fixtures | 数据怎样准备、隔离和清理？ |
| CI 配置 | 谁触发执行？失败是否阻止流水线？报告保存在哪里？ |

如果需要独立练习，可以参考 [官方 Serenity Cucumber Starter](https://github.com/serenity-bdd/serenity-cucumber-starter)。它是学习基线，复制后仍要检查版本、示例场景和构建配置。

### 8.2 版本核查：不要拼接不同年代的教程

本次检查官方 starter 的固定 commit：`0a1b17d0b24aa4d58a4be8d8901187e3b02de0eb`。

| 项目 | 该快照中的配置 | 如何使用这一信息 |
| --- | --- | --- |
| Serenity | `5.3.8` | 是 starter 快照值，不声明为全生态最新版本 |
| Cucumber | `7.34.2` | 与选定 Serenity 组合一起评估 |
| JUnit | `6.0.3` | 使用 JUnit Platform Suite；并非 JUnit 4 runner |
| BOM | `serenity-bom` | 统一 Serenity 模块依赖版本 |
| Maven test execution | Failsafe `integration-test` + `verify` | 启动 suite，并最终检查测试失败 |
| Report | `serenity-maven-plugin:aggregate` | 在 `post-integration-test` 聚合报告 |

数据来源：[固定版本 starter pom.xml](https://github.com/serenity-bdd/serenity-cucumber-starter/blob/0a1b17d0b24aa4d58a4be8d8901187e3b02de0eb/pom.xml)。

官方文档中仍常用“JUnit 5”描述 Platform Suite 集成路线；上面的 starter 快照已经配置 JUnit 6。**读教程时区分集成方式和依赖实际版本。** JUnit 6.0.3 的运行要求为 Java 17+。[JUnit 6.0.3 Overview](https://docs.junit.org/6.0.3/overview.html)

此外，这个 starter 的 compiler 配置仍写 Java 8 source/target，不能据此推断依赖能在 Java 8 上运行。本文教学 Java 代码使用 text blocks，练习工程应明确配置 Java 17 或团队选定的更高版本。

Serenity 官方文档标明：从 5.0 起 JUnit 4 支持 deprecated，Cucumber reporter 的包路径也改为 `net.serenitybdd.cucumber.core.plugin.*`。旧项目应先保持已有可工作的版本组合，迁移则单独规划。[Serenity Maven Guide](https://serenity-bdd.github.io/docs/guide/maven)、[Cucumber Configuration Reference](https://serenity-bdd.github.io/docs/cucumber/configuration-reference)

### 8.3 本文示例的目录结构

```text
project/
├── pom.xml
└── src/test/
    ├── java/com/example/payments/
    │   ├── CucumberTestSuite.java
    │   ├── steps/PaymentValidationSteps.java
    │   └── actions/PaymentApiActions.java
    └── resources/
        ├── features/payments/payment_validation.feature
        ├── junit-platform.properties
        └── serenity.conf

target/site/serenity/index.html        # 执行后默认 HTML 报告位置
```

Feature 放在 resources 中，供测试 classpath 发现。glue 要指向 step definitions 与需要的 hooks，不能只写文件路径。

### 8.4 需要哪些依赖与构建配置？

本节是**对现有工程或 starter 的配置说明，不是完整的独立 `pom.xml`**。

| Dependency | 用途 |
| --- | --- |
| `net.serenity-bdd:serenity-core` | Serenity 基础支持 |
| `net.serenity-bdd:serenity-cucumber` | Serenity 与 Cucumber 集成 |
| `net.serenity-bdd:serenity-rest-assured` | 第 9 节的 `SerenityRest` API 请求记录 |
| `io.cucumber:cucumber-java` | Java step annotations；检查现有直接或传递依赖 |
| `io.cucumber:cucumber-junit-platform-engine` | Cucumber 的 JUnit Platform engine |
| `org.junit.platform:junit-platform-suite` | Suite API 与发现执行 |
| `org.junit.jupiter:junit-jupiter-api` | 示例使用的 assertions |

如果已用 Serenity BOM，可以添加以下 API 模块；未使用 BOM 时要补上与其他 Serenity 模块一致的 version。

```xml
<dependency>
    <groupId>net.serenity-bdd</groupId>
    <artifactId>serenity-rest-assured</artifactId>
    <scope>test</scope>
</dependency>
```

在现有 `maven-compiler-plugin` 配置中将 source/target 明确设为 Java 17；这是替换旧设置，不要创建两个重复的 compiler plugin。

```xml
<configuration>
    <source>17</source>
    <target>17</target>
</configuration>
```

同时确认：

1. Failsafe 的 include 规则能发现 `CucumberTestSuite.java`，例如 `**/*TestSuite.java`。
2. Failsafe 绑定 `integration-test` 与 `verify`；不要因为 Maven 显示成功就假定执行了场景。
3. Serenity Maven plugin 与 Serenity 库版本一致，`aggregate` 绑定 `post-integration-test`。
4. 如果工程同时有 unit tests，正确分配 Surefire 与 Failsafe，避免 suite 重复执行或 unit tests 被跳过。
5. 不设置全局忽略失败；CI 必须能从退出码识别真正的测试失败。

官方 starter 中 Surefire 被跳过，因为样例采用 Failsafe 路线；不要直接把这个做法复制到需要 Surefire 执行 unit tests 的业务工程。依赖和 lifecycle 参考 [Serenity Maven Guide](https://serenity-bdd.github.io/docs/guide/maven)。

<a id="section-9"></a>
## 9. 实践示例：用 Serenity + Cucumber 验证支付校验 API

### 9.1 先明确教学契约与示例边界

我们测试的是**支付校验**，并未实现真实转账或账务处理。

假设已在测试环境准备一个独立 fixture：

```text
accountId = ACCOUNT-DEMO-001
status = ACTIVE
currency = SGD
balanceMinor = 200000                 # SGD 2,000.00
remainingDailyLimitMinor = 100000      # SGD 1,000.00
```

简化规则：`0 < amountMinor <= remainingDailyLimitMinor` 时允许；金额超限时拒绝。这里余额充足，校验是无副作用的，不扣减余额，也不占用额度。

| API | 教学契约 |
| --- | --- |
| `GET /accounts/{accountId}` | 200，返回上述 fixture 字段 |
| `POST /payment-validations` | 接收 `accountId`、`currency`、`amountMinor` |
| 合法的校验请求 | 200，返回 `decision` 和 `reasonCode`，包括业务拒绝 |
| 允许响应 | `{"decision":"ALLOWED","reasonCode":"NONE"}` |
| 超限响应 | `{"decision":"REJECTED","reasonCode":"DAILY_LIMIT_EXCEEDED"}` |

**这里选择“业务拒绝也返回 200”只为了示范 HTTP 状态与业务状态是两层含义。** 真实项目可能用 4xx 或不同响应结构，以团队的 API contract 为准。

执行前必须有符合以上契约的测试服务或本地 stub，以及 fixture。本文没有提供服务实现，也没有验证以下代码在团队项目中的编译或运行结果；这是展示完整映射关系的教学代码。

Mock / Stub 可用来验证测试接线与已模拟行为，但只对 stub 跑通不能证明真实后台正确。之后应对实际测试环境执行相应的 integration/system checks。

### 9.2 写 Feature：边界行为先清楚

文件：`src/test/resources/features/payments/payment_validation.feature`。

```gherkin
@regression @api @payment-validation
Feature: Validate payments against the remaining daily limit
  Payments must be positive and must not exceed the remaining daily limit.

  Background:
    Given a dedicated active SGD account with 100000 minor units of remaining daily limit

  Scenario Outline: Apply the remaining daily limit boundary
    When a payment of <amountMinor> SGD minor units is validated
    Then the validation decision is "<decision>" with reason "<reasonCode>"

    Examples:
      | amountMinor | decision | reasonCode           |
      | 99999       | ALLOWED  | NONE                 |
      | 100000      | ALLOWED  | NONE                 |
      | 100001      | REJECTED | DAILY_LIMIT_EXCEEDED  |
```

这一 Outline 展开为 **3 个 scenario executions**。Feature 上的 tags 作用于下属场景。Background 对每次执行检查同一份已知前置数据，而不是“整份 Feature 只执行一次”。[Gherkin Reference](https://cucumber.io/docs/gherkin/reference/)

这份 Feature 尚未覆盖非正金额、账户状态、余额不足、币种、权限和并发。**正文中的 Feature 描述不能代替这些未编写的测试。**

### 9.3 写 API Actions：封装交互，并让 Serenity 记录步骤

文件：`src/test/java/com/example/payments/actions/PaymentApiActions.java`。

```java
package com.example.payments.actions;

import io.restassured.response.Response;
import io.restassured.specification.RequestSpecification;
import net.serenitybdd.annotations.Step;
import net.serenitybdd.rest.SerenityRest;

public class PaymentApiActions {

    private RequestSpecification request() {
        String baseUrl = System.getProperty("api.base.url");
        if (baseUrl == null || baseUrl.isBlank()) {
            throw new IllegalStateException("Configure api.base.url for the test service");
        }
        return SerenityRest.given()
                .baseUri(baseUrl)
                .accept("application/json");
    }

    @Step("Read the dedicated payment-validation fixture account")
    public Response readAccount(String accountId) {
        return request()
                .pathParam("accountId", accountId)
                .get("/accounts/{accountId}");
    }

    @Step("Validate a payment amount of {1} SGD minor units")
    public Response validatePayment(String accountId, long amountMinor) {
        String payload = """
                {"accountId":"%s","currency":"SGD","amountMinor":%d}
                """.formatted(accountId, amountMinor);

        return request()
                .contentType("application/json")
                .body(payload)
                .post("/payment-validations");
    }
}
```

说明：

- 使用 `SerenityRest`，使 API 交互接入 Serenity 记录。
- `@Step` 是可读的动作记录；`{1}` 对应第二个方法参数。
- `api.base.url` 是本文代码主动读取的 Java system property，必须通过命令或工程配置传入。
- 示例固定使用合成 accountId，字符串构造用于入门；实际复杂 payload 应采用团队的 DTO/JSON serialization 方式。
- 示例契约未定义认证；对接内部 API 时按团队现有方式添加测试身份、认证和请求配置。敏感 header/body 的日志处理应在共享 request 层集中配置，并检查最终报告。

相关机制见 [Serenity API Tutorial](https://serenity-bdd.github.io/docs/tutorials/rest)。

### 9.4 写 Step Definitions：业务文字 → 动作 → Assertion

文件：`src/test/java/com/example/payments/steps/PaymentValidationSteps.java`。

```java
package com.example.payments.steps;

import com.example.payments.actions.PaymentApiActions;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;
import io.restassured.response.Response;
import net.serenitybdd.annotations.Steps;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class PaymentValidationSteps {

    @Steps
    PaymentApiActions api;

    private String accountId;
    private Response validationResponse;

    @Given("a dedicated active SGD account with 100000 minor units of remaining daily limit")
    public void verifyFixture() {
        accountId = "ACCOUNT-DEMO-001";
        Response account = api.readAccount(accountId);
        assertEquals(200, account.statusCode(), "Fixture lookup must succeed");
        assertEquals("ACTIVE", account.jsonPath().getString("status"));
        assertEquals("SGD", account.jsonPath().getString("currency"));
        assertEquals(200000L, account.jsonPath().getLong("balanceMinor"));
        assertEquals(100000L, account.jsonPath().getLong("remainingDailyLimitMinor"));
    }

    @When("a payment of {long} SGD minor units is validated")
    public void validatePayment(long amountMinor) {
        validationResponse = api.validatePayment(accountId, amountMinor);
    }

    @Then("the validation decision is {string} with reason {string}")
    public void verifyDecision(String decision, String reasonCode) {
        assertEquals(200, validationResponse.statusCode(), "Validation endpoint contract");
        assertEquals(decision, validationResponse.jsonPath().getString("decision"));
        assertEquals(reasonCode, validationResponse.jsonPath().getString("reasonCode"));
    }
}
```

`@Steps` 由 Serenity 管理 action library；`@Given` 等来自 Cucumber。实例字段保存场景上下文，避免用 `static` response 在场景间共享状态。跨多个 steps 类共享数据时，应使用团队的 scenario-scoped context/DI 方案。[Cucumber API Reference](https://cucumber.io/docs/cucumber/api/)

注意这个 Given **验证已有 fixture，并没有创建账户**。首次运行前要准备数据；真实有副作用的流程应使用独立数据和清理机制。

### 9.5 写 JUnit Platform Suite

文件：`src/test/java/com/example/payments/CucumberTestSuite.java`。

```java
package com.example.payments;

import org.junit.platform.suite.api.ConfigurationParameter;
import org.junit.platform.suite.api.IncludeEngines;
import org.junit.platform.suite.api.SelectClasspathResource;
import org.junit.platform.suite.api.Suite;

import static io.cucumber.junit.platform.engine.Constants.GLUE_PROPERTY_NAME;
import static io.cucumber.junit.platform.engine.Constants.PLUGIN_PROPERTY_NAME;

@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "com.example.payments.steps")
@ConfigurationParameter(
        key = PLUGIN_PROPERTY_NAME,
        value = "net.serenitybdd.cucumber.core.plugin.SerenityReporterParallel,pretty"
)
public class CucumberTestSuite {
}
```

这里没有在 annotation 中固定 tag filter，方便通过命令切换 smoke 与 regression。Reporter 包路径适用于 Serenity 5.x；旧版本必须使用其对应配置。[Cucumber Configuration Reference](https://serenity-bdd.github.io/docs/cucumber/configuration-reference)

### 9.6 配置串行执行与报告

文件：`src/test/resources/junit-platform.properties`。

```properties
cucumber.execution.parallel.enabled=false
```

文件：`src/test/resources/serenity.conf`。

```hocon
serenity {
  project.name = "Payment Validation - Learning Example"
}
```

入门阶段先串行执行，确认隔离和数据策略后再开启并行。如果从官方 starter 复制，该快照默认启用了 Cucumber 并行，需要明确改为 false，并检查其他配置来源。

本文 Java 代码读取的是 `System.getProperty("api.base.url")`，所以仅往 `serenity.conf` 写同名值还不够；必须传入 system property，或主动修改代码从 Serenity environment configuration 读取。

### 9.7 执行命令与应检查的结果

下面是 **Windows PowerShell 命令示例**。假设符合契约的教学服务已运行在 `http://localhost:8080`；真实测试地址替换为团队配置。

```powershell
java -version
mvn -version

# 检查 feature 与 step definition 接线，不执行场景动作
mvn clean verify "-Dcucumber.execution.dry-run=true"

# 执行本文 3 个支付边界例子
mvn clean verify "-Dapi.base.url=http://localhost:8080" "-Dcucumber.filter.tags=@payment-validation"

# 执行 regression 集合，排除未完成用例
mvn clean verify "-Dapi.base.url=http://localhost:8080" "-Dcucumber.filter.tags=@regression and not @wip"
```

Dry run 用于检查场景与步骤能否对应，**不证明 API、业务断言或数据正确**。如果 dry run 没有发现任何场景，也不能算接线通过。[Cucumber Configuration Reference](https://serenity-bdd.github.io/docs/cucumber/configuration-reference)

真实执行后，至少确认：

1. 场景数与预期一致；本文选择 `@payment-validation` 应展开为 3 次执行。
2. `Given` 确认数据成立，`When` 确实调用服务，`Then` 比较业务字段。
3. 没有 undefined、pending、unexpected skipped，且 Maven 退出码符合结果。
4. 生成的 `target/site/serenity/index.html` 对应本次执行，而不是历史报告。
5. 在独立练习分支暂时改错一个期望值，确认断言会失败；随后恢复。需要验证能捕获真实回归时，可在本地 stub 或受控测试版本引入错误行为，再确认测试失败。

不要把错误逻辑直接改进共享银行环境来演示。上述第五项是练习环境的测试有效性检查。

默认报告路径见 [Serenity Cucumber Tutorial](https://github.com/serenity-bdd/serenity-bdd.github.io/blob/master/docs/tutorials/cucumber.md)。**这里描述应有结果，并未宣称示例已经执行通过。**

### 9.8 从教学例子扩展到实际项目

下一步可以增加：非正金额、余额不足、冻结账户、角色权限、实际错误码、币种精度与日切规则。每个用例仍遵循同一条链：

```text
Confirmed rule → independent test data → action → business assertion → evidence
```

如果测试会创建支付或修改额度，需要验证副作用：例如支付记录、额度变化、事件和账务一致性。根据系统边界使用 API、事件或必要的数据库检查；数据验证范围不能超过你实际执行和观察到的部分。

<a id="section-10"></a>
## 10. 怎样把场景组织成可维护的 Regression Suite？

以下是面向后台项目的建议，不代表 DBS 内部规范。

### 10.1 Tags 是分类和选择机制

| Tag | 建议含义 |
| --- | --- |
| `@smoke` | 快速确认关键路径基本可用 |
| `@regression` | 作为约定的变更保护集合 |
| `@api` / `@ui` | 交互渠道分类 |
| `@payment-validation` / `@account` | 业务域分类 |
| `@critical` | 团队约定的重要风险类别 |
| `@wip` | 尚未完成，不作为稳定通过证据 |

标签名称由团队定义。加上 `@regression` 不会自动提升用例质量；加上 `@critical` 也不会自动设置 release gate。

下面的 smoke 命令需要项目确实已有 `@smoke` 场景；第 9 节边界例子只标记 regression 等标签，没有 smoke。

```powershell
mvn clean verify "-Dapi.base.url=http://localhost:8080" "-Dcucumber.filter.tags=@smoke and not @wip"

mvn clean verify "-Dapi.base.url=http://localhost:8080" "-Dcucumber.filter.tags=@regression and @payment-validation and not @wip"
```

这里使用 `cucumber.filter.tags` 控制执行场景。Serenity 的报告过滤是相关但独立的配置，不能仅看过滤后的报告来推断未展示的场景曾经被执行。[Cucumber API Reference](https://cucumber.io/docs/cucumber/api/)、[Serenity Filtering Reports](https://serenity-bdd.github.io/docs/reporting/filtering_reports)

### 10.2 建议的执行时机

| Trigger | 建议测试范围 | 目的 |
| --- | --- | --- |
| Local development | 受影响的 unit / component / API checks | 快速反馈 |
| Pull Request | Unit、相关 integration、短 API smoke | 发现明显问题，控制反馈时间 |
| 测试环境部署后 | Smoke，再跑重点业务 regression | 确认部署与关键规则 |
| Nightly | 更广的 regression 与所需系统交互检查 | 覆盖较慢、较多依赖的测试 |
| Release candidate | 约定的回归集合及相关人工、验收、非功能测试 | 支持交付决策 |

这是一种候选策略，没有统一要求“每次 commit 必须跑所有 UI 用例”。具体集合由运行时长、环境可用性和变更风险决定。

### 10.3 CI 流程需要保证什么？

```text
Checkout exact commit
        ↓
Resolve dependencies + select environment
        ↓
Prepare isolated data + readiness check
        ↓
Run selected tests + collect actual test count
        ↓
Aggregate Serenity report + preserve test exit status
        ↓
Archive report, machine-readable results and run context
        ↓
Triage failures / evaluate release criteria
```

应保留的上下文包括：application build、test commit、环境标识、执行时间、tag expression、场景数量、结果及报告链接。不同应用版本与测试版本的结果不应直接混为同一份证据。

Test job 失败时也尽可能保存报告。Failsafe 在 `verify` 检查测试失败，而报告若绑定 `post-integration-test`，可以在此前聚合；编译错误、启动失败等情况仍可能没有完整 Serenity 结果。CI 的产物步骤应按平台机制设置为失败时也执行。[Serenity Maven Guide](https://serenity-bdd.github.io/docs/guide/maven)

Release gate 要同时关注：**确实执行了哪些测试、未执行什么、关键失败是否未解决、环境与数据是否可信**。只有绿色图标而没有场景和覆盖上下文，不足以支撑判断。

### 10.4 数据、隔离与稳定性

- 使用已知数据。需要修改状态的用例采用每个场景独立的 ID 和账户等资源。
- 不依赖其他场景先创建数据；单条场景可以独立运行。
- 清理失败要留下证据，必要时由环境机制回收，避免污染下一轮执行。
- 共享账户、静态状态、外部额度、时间和消息顺序都可能影响并行稳定性。
- 异步状态采用有上限的 polling/await，明确等待业务条件，避免无限等待或固定 sleep。
- Stub 可模拟超时和错误；真实依赖也需要合适范围的集成验证。
- Test credentials 采用团队的 secret/configuration 机制；使用合成数据，并验证报告中的请求响应符合团队的数据处理要求。

第 9 节可以共享只读 fixture，因为契约明确校验没有副作用。真实支付接口不能自动套用这一前提。

### 10.5 测什么指标，才知道自动化有价值？

| 指标 | 需要明确的口径 | 为什么有用？ |
| --- | --- | --- |
| Key-rule coverage | 已覆盖并评审的重点规则 / 已识别重点规则 | 了解业务风险覆盖；不等于代码覆盖率 |
| Feedback duration | 从触发到可用结果的时间 | 判断是否适合 PR 反馈 |
| Execution completeness | 计划、发现、执行、跳过各有多少 | 防止“零场景绿色” |
| Flaky scenario rate | 同一版本与条件下表现不稳定的场景占比 | 衡量结果可信度 |
| Failure triage effort | 定位一次失败所花时间 | 发现维护成本 |
| Escaped defects | 在后续阶段发现的缺陷与相关遗漏 | 改进测试策略，注意多因素影响 |

这是建议的观察框架，没有给出未经团队数据支持的改善百分比。Pass rate 只描述已执行集合，不能单独代表整体质量。

<a id="section-11"></a>
## 11. 如何阅读 Serenity Report 与 Living Documentation？

默认 HTML 入口通常是 `target/site/serenity/index.html`。页面布局与状态展示随版本变化，以实际生成报告为准。

建议按这个顺序阅读：

1. **Run context**：这是哪个 application build、环境和测试版本？何时执行？
2. **Execution summary**：实际场景数、通过、失败、跳过和未完成状态。
3. **Requirements / Features**：需求与业务行为如何关联，覆盖了哪些规则？
4. **Scenario detail**：哪个 Given/When/Then 失败，前置数据是否成立？
5. **Evidence**：相关请求、响应、步骤、错误信息；UI 场景可能有截图。
6. **Limitations**：哪些需求没有场景，哪些场景没有执行，哪些下游没有观察？

Serenity 的报告和 Requirements 视图将场景与执行结果组合成 Living Documentation，可用于说明系统在已定义例子下的行为。[Serenity Reports](https://serenity-bdd.github.io/docs/reporting/the_serenity_reports)、[Living Documentation](https://serenity-bdd.github.io/docs/reporting/living_documentation)

### 11.1 为什么叫 Living Documentation？

传统说明文字可能与代码逐渐脱节。可执行规格在重跑时会暴露“预期与实际不一致”，帮助团队维护文档和实现。

但是“living”仍有条件：规则经过评审、场景保持更新、测试确实运行、结果关联到明确的版本和环境。几个月没执行的报告只能说明当时的结果；错误断言也可能一直绿色。

### 11.2 本知识库与 Serenity Living Documentation 的区别

| 资料 | 用途 | 维护触发 |
| --- | --- | --- |
| 本 Markdown 知识库 | 原理、术语、流程、学习指引 | 工具升级、团队流程变化、经验补充 |
| `.feature` 和测试代码 | 某系统的业务规格与可执行检查 | 需求、实现、接口或数据变化 |
| Serenity execution report | 特定运行的结果和证据 | 每次执行生成并保留上下文 |

这三者互相补充。知识库无需保存大量实时截图，应该链接到版本明确的测试资产与报告。

<a id="section-12"></a>
## 12. 失败怎么排查？先判断失败发生在哪一层

一条自动化测试失败，是一个需要调查的信号。可能的原因如下：

| 分类 | 表现 | 先检查什么？ |
| --- | --- | --- |
| Product defect | 前置条件成立，业务结果违反已确认规则 | 响应、业务状态、服务日志、变更 diff |
| Test defect | 断言或步骤实现错误 | Expected result 来源、映射和解析逻辑 |
| Test data problem | fixture 不存在、余额或状态不符 | 数据准备、隔离和清理 |
| Environment / dependency | 服务不可达、认证失效、下游不可用 | readiness、地址、identity、连接和依赖 |
| Expected business change | 新规则与旧期望不同 | 已评审需求、契约变更、批准的期望 |
| Flaky behavior | 条件相同时结果不稳定 | 并发、时间、异步、共享数据、外部依赖 |

建议排查顺序：**定位失败步骤 → 确认环境和前置条件 → 比对业务预期 → 独立复现 → 判断原因 → 修复并重跑相关检查**。

### 12.1 Serenity / Cucumber 常见问题

| 症状 | 可能原因 | 处理方向 |
| --- | --- | --- |
| Maven 成功，但 0 scenarios | tag 不匹配、suite 未发现、feature 路径不对、tests 被 skip | 检查数量、includes、classpath 和过滤条件 |
| Undefined step | glue 不正确或步骤文字不匹配 | 核对 package、expression 与 feature 文本 |
| Ambiguous step | 多个 definitions 匹配同一文字 | 消除重叠表达式，避免近似重复步骤 |
| Reporter class not found | 新旧 Serenity reporter 路径混用 | 按实际版本配置 plugin |
| Suite 没被 Maven 执行 | Failsafe includes 未覆盖文件名 | 查看 `**/*TestSuite.java` 及 plugin goals |
| 测试执行了但没有 HTML | reporter/aggregate 没配置，或先于测试失败退出 | 区分结果记录与 HTML 聚合 |
| IDE 正常、CI 失败 | JDK、变量、地址、数据、网络或目录不同 | 对比运行配置与上下文 |
| 单独通过、整套失败 | 状态或数据依赖、共享静态字段、清理污染 | 让场景独立，再检查并行 |
| API 200，但业务断言失败 | payload、身份、规则或返回字段不符 | 检查业务结果，而不是只看 HTTP |
| 报告出现旧结果 | 输出目录复用、历史产物混入 | 使用独立输出或执行前清理 |

配置机制可对照 [Cucumber API Reference](https://cucumber.io/docs/cucumber/api/)、[Serenity Configuration Reference](https://serenity-bdd.github.io/docs/cucumber/configuration-reference) 和 [Maven Guide](https://serenity-bdd.github.io/docs/guide/maven)。表内是常见排查方向，不代表某个错误一定由对应原因造成。

### 12.2 Flaky tests 应如何处理？

Retry 可以帮助收集信息，但“重跑绿了”不等于第一次失败已解释。保留首次失败和重试结果，调查数据、异步、环境及真实产品不稳定性。

如果必须临时 quarantine 一条不稳定场景，应记录 owner、原因、缺失覆盖和恢复条件，并保留相应风险的其他验证方式。不要用无限 retry 或长期跳过来制造通过率。

### 12.3 缺陷证据如何写？

建议包含：业务影响、环境与 build、scenario 名称、输入数据标识、步骤、expected/actual、报告链接、相关 trace/correlation ID、复现频率。Severity 描述影响程度，Priority 描述处理优先级，两者由团队判断。

<a id="section-13"></a>
## 13. 银行后台语境：重点关注什么？

下面是学习与测试设计的候选清单，是否适用取决于项目功能与架构。

| 领域 | 需要问的问题 | 测试方向 |
| --- | --- | --- |
| Monetary correctness | 精度、币种、fee、rounding、额度是什么规则？ | 边界与等价类；验证金额守恒或约定关系 |
| Authorization | 哪种角色能执行哪种动作？ | 允许和拒绝都要覆盖 |
| Maker-checker | 是否要求发起与审批身份分离？ | 同人审批、无权限审批、状态迁移 |
| Idempotency | 重复请求应得到什么结果？ | 验证不会产生多次业务效果 |
| Transaction / consistency | 失败或中断后如何维持一致性？ | 回滚、补偿、状态与账务检查 |
| Async processing | 请求接受是否等于业务完成？ | 验证最终状态与相应事件，设置等待上限 |
| Auditability | 哪些动作必须留下什么记录？ | 检查约定的 actor、action、时间和关联信息 |
| Batch / EOD | 日切、节假日、时区如何定义？ | 日期边界、重复执行、失败恢复 |
| Dependency failures | 下游超时、不可用或返回异常时怎么办？ | 错误映射、重试边界、恢复行为 |

第 9 节只验证同步校验响应，不能证明审批、账务、消息处理或审计正确。比如异步 API 返回 HTTP 202，只代表接受请求时，应继续检查契约规定的最终业务状态；具体状态码和机制以实际契约为准。

业务控制及证据要求需要与你的 QA 前辈、业务代表和项目规范对齐；本文不代替内部要求，也不推断 DBS 的实际实现。

<a id="section-14"></a>
## 14. 学习路线：从理解到能维护一条回归用例

用“能做什么”来判断进展，比只记工具名更有效。以下为建议路线，可以按项目时间压缩或扩展。

| 阶段 | 学习重点 | 完成标准 |
| --- | --- | --- |
| 1. 基础概念 | Testing、oracle、layers、Regression vs Confirmation | 能用一个业务例子解释区别 |
| 2. 测试设计 | 正向、负向、边界、决策表 | 为一个规则设计可评审的用例集合 |
| 3. BDD 协作 | Discovery、规则、例子、Gherkin | QA 与 Dev 对场景含义达成一致 |
| 4. 工具接线 | Maven、Suite、glue、steps、actions | 能解释一条 feature 如何执行到 API |
| 5. 执行证据 | Tags、断言、报告、失败排查 | 能运行一条真实测试并解释结果与范围 |
| 6. 回归维护 | 数据隔离、CI、影响分析、flaky 调查 | 将场景加入约定 suite 并持续维护 |

### 14.1 与 QA 前辈一起做的第一项实践

建议选择一个已有、稳定的后台规则，而不是从零搭整个平台：

1. 找到对应需求与现有手工用例，确认业务预期。
2. 选一个成功例子、一个拒绝例子、一个边界例子。
3. 在团队现有框架中实现，遵循现有 patterns 与 version。
4. 共同 review 数据、assertions 和覆盖范围。
5. 本地运行，验证能够捕获错误并读懂报告。
6. 按团队约定加入 regression tags 和 CI，明确维护 owner。

### 14.2 拿到内部项目后，需要补充到知识库的信息

- 实际 repository、JDK、Maven、Serenity、Cucumber 和 JUnit 版本。
- 实际测试命令、环境选择方式、认证配置与数据准备机制。
- 使用的 Action / Screenplay / Page Object pattern。
- smoke / regression / nightly 的范围与执行时机。
- CI job、报告位置、需求映射方式和失败处理 owner。

此前提到的 “Robotic Automation” 是内部项目名称，按你的说明不作框架推断；本文技术说明围绕已确认的 Java + Maven + Cucumber 展开。

<a id="section-15"></a>
## 15. 展示与讨论：向组长、QA 前辈说明什么？

**最新展示**：[6 页报告截图与兼容版 PPT](../output/Serenity_Testing_Reports_Compatible.pptx)、[逐页讲稿](../output/Serenity_Testing_Reports_Speaker_Notes.md) 与 [报告完整截图阅读指南](../output/Serenity_Report_Visual_Guide.md)。前五页使用知识结构，最后一页使用 NPA sign-off 代码示例并感谢 QA 前辈欧阳。图标与执行流程图改用直接嵌入的 PNG，架构页放大展示官方报告的步骤、页面截图、结果与耗时。概念下的说明包含含义、执行方法和具体例子。报告截图来自官方示例，实际界面随版本与配置变化，不代表 NPA 已执行结果。

展示共 6 页，细节可结合本知识库与逐页讲稿阅读。

| Page | 核心信息 |
| --- | --- |
| 1 | Testing 与 Test Cases：测试层级、前置条件、动作和断言 |
| 2 | Automation Testing：Prepare、Execute、Assert、Report |
| 3 | Regression Testing：与 Confirmation / Retest、Smoke 的关系 |
| 4 | Test Design：边界、条件组合、状态迁移和回归范围 |
| 5 | Serenity BDD：Feature、Java / SUT 与真实报告截图 |
| 6 | NPA Unit A sign-off 的操作、持久化检查与代码示例 |

可以用这段话作为开场的核心概括：

> Testing 需要先确定预期，再检查实际结果。一条 test case 写清前置条件、动作和判断依据；Automation 将检查接到代码，Regression 在变更后重测已有行为。Serenity 帮助组织执行过程和报告，具体预期仍要由需求与断言来定义。最后用 NPA sign-off 例子说明这些概念怎样接到代码里。

### 15.1 可能被问到的问题

| 问题 | 简洁回答 |
| --- | --- |
| Automation 和 Regression 是一回事吗？ | 前者是执行方式，后者是变更后的测试目的。 |
| Regression 必须自动化吗？ | 不必须；高频重复部分很适合自动化，也可能需要人工回归。 |
| 为什么还需要 QA？ | QA 提供风险视角、测试设计、探索和质量证据判断。 |
| BDD 就是写 Given/When/Then 吗？ | 还包括先协作发现规则与例子，再形成规格和自动化。 |
| Cucumber 与 Serenity 有什么区别？ | Cucumber 执行规格；Serenity 提供相关测试支持、步骤记录和报告。 |
| 为什么后台先选 API？ | 能直接观察服务规则与契约；完整流程仍需其他层级补充。 |
| 200 就算测试通过吗？ | 要继续检查业务结果及契约要求的副作用。 |
| 报告全绿就能 release 吗？ | 还要判断范围、未执行测试、环境、数据及剩余风险。 |
| 为什么不全写 E2E？ | E2E 成本与依赖较多；用不同层级分别覆盖规则与集成。 |
| 示例是否已验证 DBS 系统？ | 没有；这是虚构教学示例，下一步要映射到团队实际项目。 |
| 所有用例都要用 Gherkin 吗？ | 根据业务规格与协作价值选择，unit tests 可继续用原有方式。 |
| 第一个实际成果应该是什么？ | 一组共同评审、真实执行、能解释失败、可加入回归的场景。 |

<a id="section-16"></a>
## 16. Glossary：中英文速查

| English | 中文 / 本文含义 |
| --- | --- |
| SUT / System Under Test | 被测系统 |
| Test basis | 测试依据，如需求或接口契约 |
| Test case / Scenario | 用例 / 具体行为例子 |
| Test suite | 按目的组织的测试集合 |
| Test fixture | 已知的测试数据与前置环境 |
| Assertion | 预期与实际的检查 |
| Test oracle | 判断结果正确与否的依据 |
| Acceptance criteria | 验收标准 |
| Happy path / Negative path | 正常成功路径 / 拒绝或异常路径 |
| Boundary | 规则切换的边界 |
| Defect / Bug | 工作产物中的缺陷 |
| Failure | 系统运行时表现出的失败 |
| Confirmation / Retest | 确认原缺陷已修复 |
| Regression | 检查变更的不良影响 |
| Smoke | 快速基本可用性检查 |
| BDD | 行为驱动开发的协作方法 |
| Gherkin | 可读行为规格的语法 |
| Step definition / Glue | 把步骤文字连接到代码的方法 / package 配置 |
| Hook | 场景前后等生命周期执行逻辑 |
| Living Documentation | 与可执行规格和结果关联的文档 |
| Traceability | 需求、用例、结果与缺陷间的追踪关系 |
| Flaky test | 在相同版本和条件下表现不稳定的测试 |
| Mock / Stub | 测试中模拟协作者或响应的替身；具体区分按项目约定 |
| Idempotency | 重复同一请求不产生额外的约定业务效果 |
| CI / Continuous Integration | 持续集成 |
| Release gate | 团队设定的交付判定条件 |

<a id="section-17"></a>
## 17. Sources、维护与验证状态

### 17.1 权威来源与阅读顺序

本资料以官方文档和一手工程文章为基础。工具配置需在实际使用时重新核查；正文中的业务案例、分工表、执行策略和学习路线属于教学示范或工程建议。

| ID | Source | 主要用途 |
| --- | --- | --- |
| S01 | [ISTQB CTFL Syllabus v4.0.1](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf) | 测试概念、QA、层级、设计方法、Regression 与工具收益风险 |
| S02 | [Cucumber BDD](https://cucumber.io/docs/bdd/) | Discovery、Formulation、Automation |
| S03 | [Cucumber Example Mapping](https://cucumber.io/docs/bdd/example-mapping/) | 用 story、rules、examples、questions 澄清需求 |
| S04 | [Gherkin Reference](https://cucumber.io/docs/gherkin/reference/) | Feature、Background、Scenario Outline、Tags |
| S05 | [Cucumber API Reference](https://cucumber.io/docs/cucumber/api/) | Steps、hooks、筛选与配置 |
| S06 | [Serenity Introduction](https://serenity-bdd.github.io/docs/guide/user_guide_intro) | Serenity 的目标与生态集成 |
| S07 | [Serenity Maven Guide](https://serenity-bdd.github.io/docs/guide/maven) | 依赖、Failsafe、Maven lifecycle |
| S08 | [Serenity Cucumber Configuration](https://serenity-bdd.github.io/docs/cucumber/configuration-reference) | Suite、reporter、glue、tags、dry run |
| S09 | [Serenity API Tutorial](https://serenity-bdd.github.io/docs/tutorials/rest) | REST Assured 集成与 API actions |
| S10 | [Screenplay Fundamentals](https://serenity-bdd.github.io/docs/screenplay/screenplay_fundamentals) | Actor、Ability、Task、Question |
| S11 | [Screenplay REST](https://serenity-bdd.github.io/docs/screenplay/screenplay_rest) | API 测试的另一种组织方式 |
| S12 | [Serenity Reports](https://serenity-bdd.github.io/docs/reporting/the_serenity_reports) | 报告内容与功能覆盖视图 |
| S13 | [Living Documentation](https://serenity-bdd.github.io/docs/reporting/living_documentation) | 需求结构与文档维护 |
| S14 | [Official Cucumber Starter — pinned snapshot](https://github.com/serenity-bdd/serenity-cucumber-starter/tree/0a1b17d0b24aa4d58a4be8d8901187e3b02de0eb) | 本次核查的版本与项目结构 |
| S15 | [JUnit 6.0.3 Overview](https://docs.junit.org/6.0.3/overview.html) | Platform/Jupiter 区分与 Java 17+ 要求 |
| S16 | [The Practical Test Pyramid — Ham Vocke](https://www.martinfowler.com/articles/practical-test-pyramid.html) | 多层级测试组合的工程思路 |

初学先读 S01 的 §1、§2.2.3、§4，再读 S02、S04；工具落地时读 S07–S09，并对照团队项目。

### 17.2 知识库维护原则

- 文档、Feature 和代码随需求变更一起 review。
- 保留“已确认的规则”“教学假设”“待确认问题”的区别。
- 升级时记录 JDK、Serenity、Cucumber、JUnit 与构建 plugins 的组合。
- 为团队实践补充 owner、内部入口、报告位置与失败处理方式。
- 避免把一次执行结果写成长期保证；引用报告时标明环境、build 和时间。

### 17.3 当前验证状态

| 项目 | 状态 |
| --- | --- |
| 基础与工具资料 | 已查阅列出的公开来源，核查日期 2026-10-12 |
| 技术栈 | 用户确认 Java + Maven + Cucumber |
| 银行业务契约 | 虚构教学假设，尚未映射内部需求 |
| Java 与 Feature 示例 | 用于解释接线；未编译、未对接内部服务执行 |
| Maven 命令与报告 | 按文档和配置说明给出；没有本项目的实际执行结果 |
| PPT | 已制作 6 页详细版 `.pptx`，含说明、示例代码、可编辑流程图和逐页讲稿；实际测试代码仍未接入项目执行 |

### 17.4 Change log

| Date | Version | Change |
| --- | --- | --- |
| 2026-10-12 | 1.0 | 建立中英混用知识库：基础、自动化、回归、BDD、Serenity API 示例、CI、报告与展示建议 |
| 2026-10-12 | 1.1 | 补充 NPA 网站背景与讲稿；保留原 API 教学案例 |
| 2026-10-12 | 1.2 | 用用户模板制作 6 页详细版 PPT：前五页讲知识体系，最后一页放 NPA 代码示例和对欧阳的感谢 |
| 2026-10-13 | 1.3 | 整理发布资料，更新成品链接与展示结构，保留官方报告截图及阅读指南 |
