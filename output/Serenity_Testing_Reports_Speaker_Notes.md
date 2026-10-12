# Serenity Automation & Regression Testing：详细版逐页讲稿

对应 6 页详细版成品 PPT。每页包含解释和例子，前五页讲概念与代码结构，最后一页使用 NPA 示例。

## 1. Testing & Test Cases

Testing 就是根据需求检查软件的实际表现。它既包括运行程序，也包括需求评审、代码检查这些不运行程序的工作。运行测试时，首先要知道正确结果是什么，这个判断依据就是 test oracle。

一条 test case 需要写清前置条件、测试数据、执行动作和预期结果。Given、When、Then 可以帮助我们记住这几部分：Given 准备情况，When 执行动作，Then 检查结果。写成代码后，最后一步通常就是 assertion。操作没有报错，并不代表业务结果正确。

测试也有不同层级。Unit 检查局部逻辑，Integration 检查组件或系统怎样配合，System 检查整个系统，Acceptance 检查是否满足业务验收要求。Functional 关注做得对不对，Non-functional 关注性能、可靠性、安全性等。API 和 UI 是测试入口，不是另外一套互斥层级。

通过测试，只说明这些条件和检查项通过了。没有写到、没有执行的情况，仍然可能有问题。

参考：ISTQB CTFL v4.0.1，§1、§2.2、§3。[参考资料](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)
Gherkin：[参考资料](https://cucumber.io/docs/gherkin/reference/)

讲解时可以沿着完整 case 说下去：为什么要让其他字段合法？因为我们要确认失败来自空 name，而不是别的规则。为什么 Then 要同时检查提示和未保存？因为提示正确但记录仍被写入，也违反了需求。实际实现可通过页面重新读取或合适的 API 检查保存结果。

页面里的具体例子是必填字段 name 为空。其他字段合法、用户已登录时提交表单，预期应该出现对应的必填提示，并且数据不保存。这展示了一条完整 case 怎样从条件连接到动作和断言。示例字段与规则属于教学假设。

## 2. Automation Testing

Automation Testing 是把可以重复执行的检查接到程序和工具上。一个完整的自动化测试通常要准备数据与身份、执行操作、比较结果，再保存报告。它可以调用一个方法、发 API 请求，也可以操作浏览器。

准备数据时，要确保场景知道自己使用哪些记录，以及开始时是什么状态。执行以后，断言要检查需求真正关心的东西。例如不能只检查 HTTP 200，也要检查返回的业务状态。UI 测试也不能只看按钮点击成功，还要读实际结果。

测试代码可以分成几个部分：scenario 或 test 描述要检查什么，actions 或 client 复用操作，Page Object 处理页面定位和读取，fixture 负责数据，assertions 负责判断。这样页面定位改变时，不需要修改每条业务场景。

对 QA，自动化可以减少反复手工检查，让他们有更多时间做测试设计和探索。对项目，主要帮助是更快发现改动带来的问题。但是脚本也需要维护，环境、数据或等待条件不稳定会造成 flaky tests。重复运行直到变绿不能代替原因分析。

优先选择反复执行、规则明确、结果能可靠判断的检查。

参考：ISTQB CTFL §6。[参考资料](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)
Serenity Page Objects：[参考资料](https://serenity-bdd.github.io/docs/guide/page_objects)

四个步骤是一条可重复检查的执行过程。Prepare 的 fixture 是建立初始状态的辅助代码，不只是固定的一组数据。Execute 将具体操作封装成方法，等待应以元素可用或业务状态更新为条件。Assert 的 expected 由需求提供：假如显示成功提示，但重新读取后记录未保存，用例仍应失败。Report 的证据用于解释失败原因；flaky test 指相同检查在相同代码等条件下有时通过、有时失败。应调查数据、等待、环境或产品中的不确定行为，再决定修复，而不是不断 retry。

## 3. Regression Testing

Regression Testing 是在代码、配置、依赖或环境变更后，检查应该保持的行为是否仍然正常。它不只针对 bug fix，新功能、refactoring 和版本升级都可能需要回归。

Confirmation，也就是 retest，关注原来那个缺陷是否修好。Regression 的范围更广，要看这次修改有没有影响其他行为。Smoke 则是用短小的关键检查确认当前构建基本可用，值得继续测试。Smoke 的具体范围由团队定义。

右边的表把测试目的和执行方式分开。回归可以手工做，也可以让代码自动跑。自动化也可以检查新功能。一条原本验证新需求的测试，在以后重跑时，也可以承担 regression 的作用。

确定回归范围时，先看改了哪些模块，再检查调用方、共享逻辑和连接的系统。可以只跑相关集合，也可以按约定运行完整 regression suite。Full regression 代表团队定义的集合，不代表穷尽所有输入。

本地和 PR 通常需要较快的反馈，部署后可以先 smoke，再执行相应回归。更广的集合可以放在 nightly 或 release 检查里，具体按运行成本与风险安排。

参考：ISTQB CTFL §2.2.3、§2.3。[参考资料](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)

用一次空值 bug fix 对照三者：Retest 重跑空值场景，确认原先错误接受空值的问题被修复；Regression 同时检查合法输入仍能保存，以及共享校验逻辑的其他调用方仍正常；Smoke 先确认能登录、打开页面和完成选定的主要操作，判断构建是否值得继续测试。三个集合可以重叠，区别主要在本次执行要回答什么问题。

## 4. Test Design & Regression Scope

写自动化前，要先把 case 设计好。这里有几个常用的方法。边界值检查规则切换的地方，例如允许长度的最小值、最大值，以及边界前后一个单位。等价类则把预计行为相似的输入放在同一组，避免只是随机选很多数据。

条件组合适合用 decision table。多个条件一起决定结果时，要列出哪些组合允许、哪些拒绝，而不是只测一个成功例子。State transition testing 关注某个状态能执行哪些动作，以及动作后应该进入什么状态。也要测试不允许的转换。

Regression scope 需要结合这次修改决定。局部判断尽量在 unit tests 快速检查，API 或 integration tests 检查接口与协作，UI 和少量完整 E2E 检查用户流程和系统配合。API-first 或测试金字塔是组织测试的思路，不是固定比例。

每条场景要能独立运行。对异步结果，等待一个明确条件并设置 timeout。跑完要看实际场景数、跳过和失败原因，不能只看一个绿色图标。

参考：ISTQB CTFL §4、§5。[参考资料](https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf)
The Practical Test Pyramid：[参考资料](https://www.martinfowler.com/articles/practical-test-pyramid.html)

状态迁移检查需要覆盖操作后的结果。合法操作应该产生规定的目标状态；非法操作应该被拒绝，并保持原状态及需求要求的其他数据。决定回归范围时，要追踪改动怎样影响调用方，而不只是寻找同名文件。将局部规则放在 unit、接口协作放在 API 或 integration、少量完整路径放在 UI 或 E2E，可以减少重复又保留不同层次的检查。

页面里的数量范围 1–100 是教学规则。0 和 101 应被拒绝，1 和 100 是合法边界，2 和 99 是边界内侧的代表值。assertFalse(isValid(101)) 只示意断言，isValid 方法尚未实现。Draft 与 Submitted 的可编辑行为同样只是示例，需要按具体需求确定。

## 5. Serenity BDD Architecture

BDD 的简单理解是：先把什么情况下做什么、应该看到什么结果说清楚，再用具体例子记录下来，最后把它接到自动化代码。和 QA 或业务同事讨论例子，可以帮助我们发现规则中还没说清的地方。这对应 Discovery、Formulation 和 Automation。

Gherkin 的 feature 文件保存这些可读场景。Cucumber 解析句子，找到对应的 Java step definitions。@Given、@When、@Then 标记匹配方法，glue 指定这些方法所在的 package。

Steps 再调用复用的 actions。如果通过浏览器执行，Page Object 管理 selectors、等待和页面读取。如果调用 API，就使用相应的 client。断言由我们根据需求编写，Serenity 不会替我们决定 expected result。团队使用 Screenplay 时，也可以沿用 Tasks 和 Questions 组织相同的工作。

Maven 管理依赖和构建，JUnit Platform 与 Cucumber engine 负责发现并启动测试。Serenity 记录场景、步骤、失败和相关证据，并生成报告。CI 可以按 tags 选择 smoke 或 regression。

报告可以形成 Living Documentation，但它只说明已定义场景在明确版本、环境中的结果。场景和代码也要随需求更新。

参考：Cucumber BDD：[参考资料](https://cucumber.io/docs/bdd/)
Serenity Introduction：[参考资料](https://serenity-bdd.github.io/docs/guide/user_guide_intro)
Maven：[参考资料](https://serenity-bdd.github.io/docs/guide/maven)
Page Objects：[参考资料](https://serenity-bdd.github.io/docs/guide/page_objects)

Java 层并不自动产生正确的业务断言。Cucumber 负责将句子匹配到方法，Steps 再调用复用的操作层或 client；Page Object 集中管理定位、等待和读取，让页面结构变化时更容易维护。Assertions 根据需求判断结果，Serenity 则记录执行过程。可读场景、实现和报告各自有职责，要一起更新，报告才有解释价值。

这次用官方报告截图解释记录过程。第五页的图片裁取自官方计算器示例，图片中的操作与左上角的空 name 场景是不同示例，用于展示报告结构，不是 NPA 的执行结果。

先看 Steps：能知道实际执行了哪些动作。再看 Screenshots：缩略图关联当时的被测页面，完整原图还展示了计算器页面的截图预览。Outcome 显示每个记录步骤的结果，最右侧显示耗时。图中的 SUCCESS 属于这个官方示例；不能把它理解成我们的项目已经通过测试。截图捕获频率由配置决定，并不是任何 Java 方法、任何执行方式都自动有浏览器截图。

图标、流程图与截图配合后，可以这样讲：Feature 定义要检查的行为，Cucumber / Java 操作 SUT 并执行断言，Serenity 把已记录的场景和步骤汇总到报告。测试预期仍由需求和我们的断言定义。你们用 Cucumber 时，场景和 Gherkin 步骤接入 Serenity 的集成后才能生成相应报告；子步骤可以通过 instrumented step libraries 等方式记录。

图片来源：Serenity 官方 Step Libraries 文档。文档截图包含旧版本界面；实际项目的样式和可用证据会随版本、集成与配置变化。
[参考资料](https://serenity-bdd.github.io/docs/guide/step_libraries)
[参考资料](https://serenity-bdd.github.io/assets/images/calculator-step-screenshots-f155907793e985d481afe765cc342085.png)
截图配置参考：
[参考资料](https://serenity-bdd.github.io/docs/reference/serenity-properties)
报告总览与结果列表：
[参考资料](https://serenity-bdd.github.io/docs/reporting/the_serenity_reports)

左边的 Gherkin 片段与第一页的空 name 例子对应。Cucumber 不会自动理解字段校验的业务含义，需要通过 glue 匹配 step methods。下方架构图表示可读场景进入 Java 执行，再由 Serenity 记录结果。SUT 就是被测系统。箭头表达执行流程，场景记录在整个运行过程中发生。

## 6. NPA Sign-off Example

最后用一个 NPA sign-off 例子把前面的概念连起来。假设我们准备了一个 Pre-NPA proposal，Unit A 的 sign-off 还是 Pending，测试用户有相应权限。这个前置状态需要由数据准备和登录步骤建立。

When 对应提交 Unit A 的 sign-off。Then 重新打开同一个 proposal，读取对应 unit 的状态，再跟 expected status 比较。这样检查的是保存后的结果，不只是按钮点击或成功提示。

代码是局部示意。actions.submitFor 需要按实际页面或 API 实现，page.reopen 和 statusFor 应处理页面加载并定位正确记录。SIGNED_OFF 是预期值的示意，真实状态名称和权限规则要根据项目确认。它没有表达“签完以后自动进入下一阶段”这样的假设。

以后 sign-off 的相关实现修改了，就可以重跑这一场景做 regression，再根据改动增加其他相关角色、权限和阶段规则的检查。

感谢欧阳这位 QA 前辈的指导和帮助。

NPA 背景来自用户描述。方法名与状态值为教学示例，未接入项目运行。Serenity 集成参考：[参考资料](https://serenity-bdd.github.io/docs/tutorials/first_test)
Cucumber Steps：[参考资料](https://cucumber.io/docs/cucumber/api/)

最后一页可按代码顺序讲解：先用独立 proposal 和有权限的用户建立前置条件，proposalId 保证操作与断言指向同一条记录。When 执行 Unit A sign-off；Then 重新打开该 proposal，读取 Unit A 的状态并比较预期。重新打开是为了验证保存后的状态。页面读取方法应等待加载完成，否则可能读到旧状态。这个示例只检查一个 unit 的保存，不推断整个 proposal 的下一阶段。
