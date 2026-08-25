# 0
python
、
https://walkinglabs.github.io/learn-harness-engineering/zh/projects/project-01-baseline-vs-minimal-harness/
https://walkinglabs.github.io/learn-harness-engineering/zh/resources/
https://swarmskills.openjiuwen.com/skills/c7dc8f03a5df49d78d2615f888495b11?version=1.0.0
https://javaguide.cn/ai/agent/harness-engineering.html#harness-%E5%88%B0%E5%BA%95%E6%98%AF%E4%BB%80%E4%B9%88
https://keras.io/examples/

https://datawhalechina.github.io/easy-vibe/zh-cn/stage-1/learning-map/
https://islinxu.github.io/claude-code-book/vol1/ch01_%E4%BB%80%E4%B9%88%E6%98%AFAI_Agent.html

https://www.modelscope.cn/datasets
https://www.modelscope.cn/datasets

https://bot.n.cn/
https://www.computervision.zone/

https://andersbrownworth.com/blockchain/blockchain
https://teachablemachine.withgoogle.com/

[https://teachablemachine.withgoogle.com/](https://classroomscreen.com/app/t/local-only-team-id/sd/ae982bb1-4af5-47fd-b03a-b9dc1a2bd0ec/s/9399fd1e-1b09-4296-9980-f1d96f2a392c)
https://poloclub.github.io/

https://github.com/datawhalechina/leedl-tutorial
https://zh-v2.d2l.ai/
https://www.cs.utoronto.ca/~fidler/teaching/2018/CSC2548.html
https://d2l.ai/chapter_preface/index.html


客户侧：银行软开部门
环境：交行：客户侧自身环境、工行：agentarts
场景：手机银行app（产品筛选、产品解读（主-从agent架构）
输入：用户投资问题
输出：理财、保险产品列表（筛选）+理财、保险产品产品解读（解读）+兜底回复
现状：我说的你不懂，你说的我没用
整体方案场景验证流程
1、解决认知对齐问题，客户看见：定义单条静态工作流，创意想法到demo原型，明确量化标准
2、解决功能实现问题，系统跑通：需求到Agent工作流开发，先跑通。搭建多个Agent工作流，相互跳转，主从工作流架构实现
3、解决质量稳定问题，上线放心：开发态工作流到生产态工作流。引入单层AI控制器（意图识别），体系化的评测集，上下文管理优化、人工兜底通道、准确率和时延在生产可接受范围
4、解决规模效率问题，经验增值：引入多个AI控制器，标准化资产，沉淀行业知识


手机银行app方案：
主agent：意图识别，是产品筛选还是产品解读还是兜底回复
产品筛选：手机银行投资顾问，收到用户关于四类金融产品（基金、保险、理财、黄金）的投资问题，产品类别识别以及条件提取，禁止直接回答问题。
子agent，将用户问题生成筛选条件和状态值，格式均为json，每层均有确定工具避免幻觉。如有涉及问题不清晰，追加话术模板多轮对话确认。 
输出4个状态，0是有召回，需要追问；1是无补充，筛选条件满足；2是无任何有效筛选，追问；3是无关问题回复。
RAG涉及追加话术、产品数据（用于无关问题回复）、列表检索话术
调用客户ESB数据库，调用银行历史数据和产品池

产品解读：子agent
用户-RAG检索产品名和意图-大模型提取标签转换json格式-手机银行API调用产品数据+RAG检索回复话术库-组装回复信息-回复用户


提示词优化：                              角色定位 → 筛选条件体系 → 核心任务 → 交互规则 → 输出格式 → 注意事项
• 初始阶段（第1轮）
•         ├── 角色定位 + 基础分类
•     条件提取阶段（第2轮）
•         ├── 增加筛选条件提取
•     交互系统阶段（第3轮）
•         ├── 建立多轮对话状态管理
•     完整系统阶段（第4轮）
•     ├── 完善条件体系
•     ├── 细化交互规则
•     ├── 标准化输出格式
•     └── 增加边界处理
• RAG优化策略
• 1、提示词优化；2、检索召回，混合与索引，关键词+向量语义检索；3、重排序，引入重排序模型，对TOP-k结果进行二次相关性打分；4、生成回答，融合与溯源，提示词+提示词约束模型，抑制AI幻觉
• 意图识别模型微调：SFT，LORA，基于LLaMA-Factory的框架，训练数据1000+到5000+，准确率从73%提升到97%，长尾意图识别准确率不足30%到90%。Qwen2.5 1.5B，Ascend910B，训练10 epoch，BS 4 
• 数据工程：基础数据+情境化语句+badcase长尾覆盖
主控数据，5k，评测集1400；产品筛选，7k，评测集400，产品解读，4000，评测1000
• 数据构建：业务侧精标、软开侧制作，1k+高质量数据——大模型生成——三轮迭代优化，质量评估+人工筛选
	• 迭代思路：基础表达——情景化语句（产品解读：不同年龄段用户表达方式；产品筛选：多种表达方式）——长尾场景，针对badcase设计问题提升准确率
• 最终效果：精度95%以上，需要大量合成数据；时延：Agent时延要再300-500ms
	• 安全：大模型生成内容不能直接面客、agent平台单独架构，输出判断
	• 可拓展+可靠性要求
• 优化：精度：数据+微调、推理优化、智能体优化
	• 时延：模型（小模型+减少输出token，减少输入token，MindIE推理框架，双卡张量并行）+平台（节点优化、网关优化）+整体：（减少模型调用次数+降低复杂度+同城部署）
• 



