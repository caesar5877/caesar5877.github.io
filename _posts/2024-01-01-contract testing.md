合同测试的编写是基于**消费者驱动方法**（Consumer Driven approach）实施的 [1, 2]。这种方法将编写和验证合同的职责明确地分给了两个角色：**消费者侧**负责定义期望并生成合同文件，而**提供者侧**负责验证是否满足这些期望 [1, 2]。

以下是详细的合同测试编写步骤和要求：

---

## 1. 消费者侧：编写合同（定义期望）

消费者侧的职责是生成一个或多个 **pact 文件**，这些文件定义了消费者角度所需的 API 接口依赖关系 [1, 2]。消费者将这些 pact 文件发布到**共同的 PactFlow Broker** 供提供者验证 [1, 2]。

### A. 定义合同范围和内容

编写合同时，消费者需要包含以下要素：

1.  **端点本身** [1, 2]。
2.  **所有请求参数**（名称和数据类型） [1, 2]。
3.  **所有预期的响应字段**及其精确的名称和数据类型 [1, 2]。

**重要说明：** 消费者**不需要**包含提供者 API 产生的所有字段，只需列出消费者依赖的响应字段即可 [1-4]。

### B. 消费者合同代码实现（以 Java 类为例）

在消费者代码中（例如 `TransactionSearchUIRESTConsumerContractTest.java` [5, 6]），使用特定注解来定义合同结构：

*   **测试场景定义：** 使用 **`@PactTest` 注解**，基于每个 API 端点定义特定的测试案例场景 [5, 6]。
*   **合同内容定义：** 使用 **`@Pact` 注解**定义实际的合同 [5, 6]。
    *   这是消费者定义其对提供者期望的地方，包括预期的端点 URL、请求以及所需的各个字段名称和数据类型 [5, 6]。
*   **方法名称匹配：** 被 `@Pact` 注解标记的方法名称，必须与 `@PactForTest` 注解的 `pactMethod` 参数**完全匹配** [5, 6]。

### C. 定义请求和预期响应

一个定义合同的方法（例如 `pactGetDepositTransactionWhenIdExists`）详细描述了 API HTTP 请求头部以及响应的期望 [7, 8]：

1.  **请求定义：**
    *   定义被配置的**端点名称** [7, 8]。
    *   API 端点请求的**路径**（使用 `.path` 方法） [7, 8]。
    *   API 端点请求中包含的**任何请求参数**（使用 `.matchQuery` 方法） [7, 8]。
    *   API 端点请求中要求的**头部字段**（使用 `.matchHeader` 方法） [7, 8]。
    *   API 端点请求的**预期响应**（使用 `.body` 方法） [7, 8]。
2.  **响应体定义：**
    *   响应体通常在一个单独的方法中定义（例如 `getExpectedDepositTransactionResponseBody`） [3, 4]。
    *   这个方法定义了 API HTTP 响应体的合同，仅包含 UI-REST 层所依赖的字段 [3, 4]。
    *   对于 **基本数据类型**（包括 `stringType`、`integerType`、`decimalType`、`booleanType`），必须包含一个**示例值** [3, 4]。这个示例值需要是指定数据类型的**有效值** [3, 4]。
    *   **子对象数组**由数组数据类型表示，为了清晰，其细节通常会分离到单独的实用程序文件中 [3, 4]。

---

## 2. 提供者侧：编写验证（满足合同）

提供者合同测试案例（例如 `TransactionSearchControllerContractProviderTest.java` [9, 10]）不需要列出 API 产生的所有字段 [1, 2]。它的主要任务是从 PactFlow Broker 检索 pact 文件，执行验证，并将验证结果报告回 Broker [1, 2]。

### A. 定义提供者状态

提供者代码使用 **`@State` 注解**来定义基于每个 API 端点的测试案例场景 [9, 10]：

*   **唯一标识符：** `@State` 注解括号内的值必须是**唯一的**，因为它代表提供者产生的一个唯一的 API 端点 [9, 10]。消费者使用这个唯一值来识别要验证的 pact 文件中的端点 [9, 10]。
*   **设置 Mock 调用：** 被 `@State` 标记的方法设置了对底层服务（例如 `DepositTransactionService`）的模拟调用 [11, 12]。

### B. 建立 Mock 响应的要求

提供者必须设置满足合同要求的模拟数据：

*   模拟调用返回一个**完整的样本 JSON 响应对象**（例如 `DepositTellerMonetaryTransactionEntity`） [11, 12]。
*   返回的 JSON 数据必须包含**每一个字段和每一个子对象字段**，并且这些值**不能为 null** [11, 12]。
*   即使 API 端点允许字段值为 `null`，出于验证目的，返回的 JSON 也必须包含该字段的有效**非 null 值** [11, 12]。
*   **子对象数组**必须包含**至少一个元素**，且每个字段都必须存在且非 null [11, 12]。

---

## 3. 本地测试运行步骤

在本地运行合同测试通常遵循以下顺序步骤 [13, 14]：

1.  **生成 Pact 文件（消费者侧）：** 在消费者侧（`ctp-transaction-ui-rest`）运行感兴趣的测试案例（全部或子集），生成包含端点内容的 pact 文件 [13, 14]。成功后，pact 文件（例如 `110417__cipteller__ctp-transactions-ui-rest-110417__ctp-transactionsearch-api.json`）将生成在 `target/pacts` 文件夹中 [13, 15, 16]。
2.  **手动复制 Pact 文件：** 将第一步生成的 pact 文件手动复制到提供者侧（`ctp-transactionsearch-api`）的 `test/resources/pacts` 文件夹中 [13, 14, 17, 18]。
3.  **运行提供者验证：** 针对手动复制的 pact 文件运行单个提供者侧合同测试案例 [13, 14]。
4.  **临时代码修改（本地验证）：** 为了让提供者针对本地复制的文件运行，需要在提供者测试类 (`TransactionSearchControllerContractProviderTest.java`) 中进行临时更改 [19, 20]：
    *   **注释掉** `@PactBroker` 注解 [19, 20]。
    *   **取消注释** `@PactFolder` 注解（并导入所需文件） [19, 20]。
    *   *注意：不应将这些更改推送到版本控制中* [19, 20]。
5.  如果测试案例成功执行，测试运行结果将显示一个**绿色复选标记**，确认已验证的端点 [19, 20]。

*若要验证 PactBroker 上已发布的文件，则需取消注释 `@PactBroker` 注解并注释掉 `@PactFolder` 注解 [21, 22]。*

---

**总结的比喻：**

编写合同测试就像是**餐厅（消费者）**向**食品供应商（提供者）**下达详细的订单。餐厅只在订单（pact 文件）中列出它做菜**必须**需要的食材（字段名称、数据类型和数量），而不需要知道供应商仓库里所有其他的存货。供应商收到订单后，会检查自己的库存（验证代码）和流程，确保他能够**完全满足**订单上列出的每一项要求，即使他实际生产的比订单要求的多。



下面是你上传的两张图片内容的完整解析与结构化整理，我已将杂乱的截图内容转成清晰、可直接用于系统设计文档或面试讲解的结构化文本。

⸻

✅ 第一部分：管理验证状态流程（Verification Status Flow）解析

系统需要处理以下几种验证状态：

⸻

1. VERIFIED（已验证）

动作：
	•	生成一次性令牌（OTT）
	•	更新状态为 ISSUED
	•	返回成功状态 SUCCESS_DVC_TRUST

⸻

2. DENIED（拒绝）

动作：
	•	设置为 DENIED 状态
	•	记录验证失败

⸻

3. FAILED（失败）

动作：
	•	设置为 DENIED 状态
	•	记录系统失败（例如内部错误）

⸻

4. EXPIRED（过期）

动作：
	•	检查令牌是否超时
	•	更新为 EXPIRED 状态

⸻

5. PENDING（待处理）

动作：
	•	生成新的中继令牌（intermediate token）
	•	设置重试等待时间
	•	继续轮询（polling）

⸻

✅ 第二部分：handle() — 主处理方法解析

handle() 方法的主要逻辑步骤如下：
	1.	解密中继令牌
	2.	验证中继令牌有效性
	3.	从缓存或数据库获得验证详情
	4.	从风险验证响应中解析验证状态
	5.	检查令牌是否已过期
	6.	根据验证状态处理：
	•	VERIFIED → 生成 OTT，返回成功
	•	DENIED / FAILED → 返回拒绝
	•	EXPIRED → 返回过期
	•	PENDING → 重新存储中继令牌，继续轮询
	7.	调用风控逻辑（例如 RBA / RBAAD）
	8.	发送审计事件

⸻

✅ 第三部分：关键配置解析

通过 PollingServiceConfigProperties 配置的属性：
	•	令牌过期时间
	•	OTT 过期时间
	•	首次轮询窗口
	•	重试等待时间
	•	风险验证开关
	•	CIU（Customer Identity Update）发布开关

该类方法 modifyDLResponse() 用于根据风险验证响应动态修改驾驶证（Driver License）验证状态。

⸻

✅ 第四部分：主要功能（Core Logic）解析

⸻

1. 获取风险验证响应

从 challengeStatusDo 中读取：
	•	riskVerificationResponse
	•	verificationStatusResponse

⸻

2. 判断验证成功情况（VERIFY_SUCCESS）

两种成功的来源：

(A) SUCCESS_DVC_TRUST
	•	当 IDVAAS 返回成功
	•	且 CFM 返回 CompleteChallenge_AllowTxn
	•	→ 设定为设备信任成功

(B) SUCCESS
	•	当 IDVAAS + CFM 都为 CompleteChallenge 类型
	•	→ 设定为普通验证成功

DENIED
	•	其他情况均视为拒绝

⸻

3. 判断是否需要重新发起验证（VERIFY_RECHALLENGE）

动作：
	•	更新状态为 RE_CHALLENGED
	•	更新下一步轮询时间
	•	保存更新后的状态

⸻

4. 判断验证失败情况（VERIFY_FAILURE）

动作：
	•	设置验证失败
	•	记录系统失败

⸻

🔥 第五部分：特殊处理

针对某些特定的渠道（如 dLvcTrustExcludedChannels）：

即使应该返回 SUCCESS_DVC_TRUST，也会统一降级为 SUCCESS。

这是为了保持渠道之间的兼容性与一致行为。

⸻

