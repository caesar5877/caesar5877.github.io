![sfe](cache-ECS.png)
第一部分：AWS 基础与身份管理

AWS 区域、可用区与数据中心
AWS 成本与支持
AWS IAM（身份与访问管理）角色
CCB EAC（环境即代码）与 JPM C Atlas Terraform
Spinnaker 在 AWS 部署中的使用
Challenge Cache 服务的 IAM 角色创建代码
第二部分：AWS 网络与计算服务

AWS Route 53
AWS NLB（网络负载均衡）
AWS ALB（应用负载均衡）
AWS Lambda
AWS SNS
AWS EventBridge
AWS ECS（使用 Fargate 的无服务器计算）
Challenge Cache 从 GAP 迁移到 AWS ECS Fargate 的配置与演示

IAM Role 通过 Security Token Service 提供临时访问权限，实现安全的跨账户与联合身份访问。
这页讲的是 **通过 CCB EAC 框架创建 AWS IAM 角色的流程**。核心逻辑如下：  

1. **CCB EAC（wrapper）**  
   - 企业内部封装层，用于标准化角色创建请求。  
   - 负责接收输入（如角色名、策略、信任关系）。  

2. **Atlas JPMC Terraform（wrapper）**  
   - JPMorgan 内部对 Terraform 的再封装。  
   - 将 EAC 的输入转换为 Terraform 模块参数。  

3. **Terraform**  
   - 基础设施即代码工具（IaC）。  
   - 根据定义文件生成 IAM Role、Policy 等资源。  

4. **AWS**  
   - 最终在 AWS 环境中创建 IAM 角色。  

**总结：**  
整个流程是一个多层封装的自动化管道：  
**CCB EAC → Atlas Terraform → Terraform → AWS**  
开发者只需在 EAC 层提交请求，底层由自动化框架完成角色创建与部署。
