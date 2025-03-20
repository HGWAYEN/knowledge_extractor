# knowledge_extractor
群聊总结  基于关键词匹配，引入评分机制进行语义增强，也可启用LLM模型

# 起源

因为群友太能分享，动不动999+，因此想到了使用AI进行总结。干中学！

先想出初版prompt（抱歉没找到orz）

主要思路就是先用关键术语做匹配，再评分。  这一步不需要用到大模型    后面再根据输出结果引入 减少成本

经过米特特改良后的第一版promot

```plain
[TASK]
基于标签优先策略的多源聊天记录知识萃取与结构化重组
[PROCESS_FLOW]
文件解析 → 时序重建 → 标签检测 (强制) → 语义增强 → 跨会话关联 → 知识拓扑生成
[INPUT_PARAMS]
{
chat_txt: """
[文件规范]
时间戳格式: YYYY-MM-DD HH:mm:ss
用户标识：连续无空格字符串
内容段：允许换行的自然语言
[示例片段]
2025-01-21 09:32:47 user1
申请大数据平台是做啥德
2025-01-21 09:33:33 user2
这个是啥意思
"""
}
[OUTPUT_FORMAT]
markdown
# 知识图谱构建报告

## 核心知识单元
🟢 带标签知识块
• [#知识分享] [技术领域] @发起用户
  ├─ 知识密度评分: ★★★☆ (基于信息熵计算)
  └─ 关联知识点: [同类技术] > [依赖组件]

⚪ 潜在价值内容
• [隐含技术点] @发现用户
  ├─ 置信度: 82% (NLP语义分析)
  └─ 推荐验证: [测试方法]@[工具链]

## 知识关系拓扑
```mermaid
graph LR
    A[#知识分享] --> B({{具体技术点}})
    B --> C[[实施路径]]
    D[未标记内容] -.-> B
    C --> E{验证指标}

[CONSTRAINTS]
标签检测使用双重机制：
显式标签：# 知识分享
隐式特征：包含≥3 个专业术语的消息
未标记内容需满足以下任一条件方可提取：
包含 CVE / 模型名称等实体
对话深度≥3 轮次 (含追问)
涉及防御方案设计模式
输出包含原始时序标记：[2025-01-21 09:32]
智能增强说明：
时序重建引擎：自动校正错位时间戳 (误差 < 15s)
语义补偿机制：修复 "做啥德" 等方言变体
领域自适应分类器：
网络安全置信度阈值: 0.78
AI 安全动态特征库：含 LLM 攻击链模式库
知识拓扑验证：自动匹配 MITRE ATT&CK 框架
该版本强化：
・方言容错处理
・时序依赖性保持
・显隐式知识联合挖掘
・符合知识图谱 ISO/IEC 21838 标准
```



使用cursor，启动面向AI编程。在这过程中根据结果，不断进行优化，比如删除了知识关系拓扑概念，因为没有引入AI效果不是很好 doge。

具体就是每次提出多个待优化的点，然后交给cursor完成。

最后得到了比较满意的效果。

一个是md文档，大纲如下：  
![](https://cdn.nlark.com/yuque/0/2025/png/42869913/1742443776120-eecf10b7-3693-4834-94a4-175212309310.png)



也编写了web页面。

![](https://cdn.nlark.com/yuque/0/2025/png/42869913/1742443844050-65063a16-424d-4387-91b5-fe213a0c6d0b.png)



“潜在知识讨论单元”展示

![](https://cdn.nlark.com/yuque/0/2025/png/42869913/1742444305567-b3f85929-7ee0-4d8f-b81c-ddca80d84b50.png)



> PS：关于聊天记录，使用的是memotrace这个导出工具。



主要代码执行逻辑如下：

## <font style="color:rgb(51, 51, 51);">执行逻辑</font>

<font style="color:rgb(51, 51, 51);">该代码实现了一个基于聊天记录的知识提取系统，主要通过以下步骤进行知识提取：</font>

1. **<font style="color:rgb(51, 51, 51);">初始化</font>**<font style="color:rgb(51, 51, 51);">：创建 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">ChatKnowledgeExtractor</font>`<font style="color:rgb(51, 51, 51);"> 类的实例，加载配置文件和初始化相关组件（如分词器、LLM API 管理器等）。</font>
2. **<font style="color:rgb(51, 51, 51);">处理聊天记录</font>**<font style="color:rgb(51, 51, 51);">：调用 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">process</font>`<font style="color:rgb(51, 51, 51);"> 方法，传入聊天记录文本，进行以下处理：</font>
   - **<font style="color:rgb(51, 51, 51);">解析聊天记录</font>**<font style="color:rgb(51, 51, 51);">：使用 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">parse_chat_file</font>`<font style="color:rgb(51, 51, 51);"> 方法将聊天记录解析为结构化数据（消息列表）。</font>
   - **<font style="color:rgb(51, 51, 51);">时序重建</font>**<font style="color:rgb(51, 51, 51);">：调用 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">rebuild_timeline</font>`<font style="color:rgb(51, 51, 51);"> 方法，校正消息的时间戳，确保消息按时间顺序排列。</font>
   - **<font style="color:rgb(51, 51, 51);">标签检测</font>**<font style="color:rgb(51, 51, 51);">：使用 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">detect_tags</font>`<font style="color:rgb(51, 51, 51);"> 方法检测消息中的显式和隐式标签，提取专业术语和关键词。</font>
   - **<font style="color:rgb(51, 51, 51);">语义增强</font>**<font style="color:rgb(51, 51, 51);">：调用 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">semantic_enhancement</font>`<font style="color:rgb(51, 51, 51);"> 方法，使用语义模型增强消息的领域得分和知识评分。</font>
   - **<font style="color:rgb(51, 51, 51);">跨会话关联</font>**<font style="color:rgb(51, 51, 51);">：使用 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">cross_session_correlation</font>`<font style="color:rgb(51, 51, 51);"> 方法提取知识单元和潜在价值内容。</font>
   - **<font style="color:rgb(51, 51, 51);">生成知识拓扑</font>**<font style="color:rgb(51, 51, 51);">：调用 </font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">generate_knowledge_topology</font>`<font style="color:rgb(51, 51, 51);"> 方法生成知识图谱报告。</font>
3. **<font style="color:rgb(51, 51, 51);">输出报告</font>**<font style="color:rgb(51, 51, 51);">：将生成的知识图谱报告保存到指定的输出文件中。</font>

