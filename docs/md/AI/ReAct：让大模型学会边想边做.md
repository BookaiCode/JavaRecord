传统聊天机器人相信大家都用过——你问一句，它答一句。线性，简单，但遇到复杂问题就露馅了。比如问"特斯拉股价相比去年涨了多少"，它要么瞎编，要么说"我无法获取实时信息"。

2022年，Google Research提出了ReAct框架。它要解决的就是这个问题：**让大模型像人一样，一边想一边做，做完再看结果，接着想下一步**。

## ReAct的核心原理

### 先看个例子

想象你在一个陌生城市旅行。

早上醒来，你想：今天天气怎么样？要不要带伞？

你打开天气APP看了一眼——有阵雨。

于是你调整计划：上午去博物馆躲雨，晚上再去看夜景。

这个"想→做→看→再想"的过程，就是ReAct在做的事。

### 三个阶段

ReAct由三个部分构成：

**思考（Thought）**：分析当前问题，决定下一步做什么。比如"用户问的是某国人口，我需要查数据"。

**行动（Action）**：调用外部工具。输出格式类似`search(query="某国人口")`。

**观察（Observation）**：工具返回结果，成为下一轮思考的依据。比如`{"population": "1.4亿"}`。

循环往复，直到任务完成。

![ReAct核心循环机制](https://mmbiz.qpic.cn/mmbiz_jpg/55IJsnyicJucTmlsa0o2IQt2CkKl8g09J9KBMphfOBZPooZcexoNRkgWToSwH6X0AKARVcZpB6r4pzBVlbNUrIEOibKCcY1pzUPEViaibkm9AN8/640?wx_fmt=jpeg&from=appmsg)

### 什么时候停下来

两种方式：

- **硬限制**：设个最大迭代数，比如`max_iterations=10`，到了就强制结束。
- **条件触发**：模型觉得自己很有把握了，或者连续失败好几次，就主动收手。

## 技术实现

### 工具怎么设计

三个原则：

**原子性**：一个工具只做一件事。计算器就做计算，搜索就做搜索，别搞大而全。

**强契约**：用JSON Schema定义清楚输入输出格式：

```json
{
    "name": "get_weather",
    "description": "查询指定城市的天气信息",
    "parameters": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "城市名称，如北京、上海"
            }
        },
        "required": ["location"]
    }
}
```

**安全第一**：删数据、转账这类敏感操作必须有权限控制。

### 提示词怎么写

ReAct提示包含四个部分：

- 要求模型展示思考过程。
- 告诉它有哪些工具可以用。
- 提示它在每个操作后重新评估。
- 设定循环退出条件。

### 从提示词到原生工具调用

早期做法是在提示词里格式化输出：

```
思考：我需要查询北京的天气
行动：get_weather(location="北京")
观察：{"temperature": "25℃", "condition": "晴"}
```

问题很明显：模型可能"废话连篇"，格式也可能乱套。

后来有了**原生工具调用**，直接微调模型让它输出结构化JSON。稳定多了，还能并行执行多个工具。

![工具调用流程图](https://mmbiz.qpic.cn/sz_mmbiz_jpg/55IJsnyicJue56U8IQnzVrFPBwEROrkwv4psGgPBNQX33hzADRj5LQ9iax14uSWMGQ2NyFB5n0mCk7Y3MOhCiaYFSNDlkU9mCvtSeuWbpEklqA/640?wx_fmt=jpeg&from=appmsg)

### 在Java中使用ReAct：LangChain4j示例

如果你是Java开发者，可以用 LangChain4j 快速实现 ReAct。核心思路是：用 `@Tool` 注解定义工具，框架自动处理推理-行动-观察的循环。

**第一步：定义工具**

```java
import dev.langchain4j.agent.tool.Tool;

public class WeatherTools {
    
    @Tool("获取指定城市的实时天气信息")
    public String getWeather(String location) {
        // 实际项目中调用天气API
        return switch (location) {
            case "北京" -> "晴天，15℃，空气质量良好";
            case "上海" -> "多云，18℃，有轻度雾霾";
            default -> "未找到该城市的天气信息";
        };
    }
    
    @Tool("查询股票实时价格")
    public String getStockPrice(String stockCode) {
        // 实际项目中调用股票API
        return "股票" + stockCode + "当前价格：168.5元，涨幅+2.3%";
    }
}
```

**第二步：创建助手接口**

```java
import dev.langchain4j.service.AiServices;

public interface Assistant {
    String chat(String userMessage);
}
```

**第三步：构建并使用**

```java
import dev.langchain4j.model.openai.OpenAiChatModel;

public class Main {
    public static void main(String[] args) {
        // 配置模型
        OpenAiChatModel model = OpenAiChatModel.builder()
            .apiKey("your-api-key")
            .modelName("gpt-4")
            .build();
        
        // 构建助手，绑定工具
        Assistant assistant = AiServices.builder(Assistant.class)
            .chatLanguageModel(model)
            .tools(new WeatherTools())  // 注册工具
            .build();
        
        // 提问——框架会自动执行ReAct循环
        String answer = assistant.chat("北京今天天气怎么样？适合户外运动吗？");
        System.out.println(answer);
    }
}
```

**运行时发生了什么？**

当用户问"北京天气怎么样"时，LangChain4j 内部会：

1. **思考**：模型分析问题，决定调用 `getWeather` 工具。
2. **行动**：执行 `getWeather("北京")`。
3. **观察**：得到 "晴天，15℃，空气质量良好"。
4. **再思考**：模型结合天气数据，判断适合户外运动。
5. **最终答案**：返回完整回答。

整个过程对开发者透明，你只需定义工具，剩下的交给框架。

**两种模式的区别**

| 模式 | 适用场景 | 特点 |
|-----|---------|------|
| 函数调用（推荐） | OpenAI、Claude等支持工具调用的模型 | 稳定可靠，直接输出结构化调用 |
| 文本解析 | 开源模型、不支持工具调用的模型 | 通过提示词让模型输出 `Action: xxx` 格式，再解析 |

LangChain4j 会根据模型类型自动选择模式。如果你的模型支持函数调用，优先用这个——更稳定，不容易出错。

## 为什么ReAct管用

![ReAct与CoT对比](https://mmbiz.qpic.cn/mmbiz_jpg/55IJsnyicJufm0JRAfdqE7An7PmO7eAHQZrGw3aJp2YvibZRxMmybR0tSKHtHRaic9V7KtG9o53Bvzygz0jeG2PpOz0VfHBvFiaffNmiaV8I3l6Q/640?wx_fmt=jpeg&from=appmsg)

### 减少胡说八道

假设用户问："今天A股涨得最多的是哪只？"

普通模型可能直接猜"茅台"——反正训练数据里有。

ReAct模型会：
1. 想到：我得查实时数据。
2. 调用股票API。
3. 看结果：涨最多的是XXX。
4. 根据真实数据回答。

不靠记忆，靠查证。

### 出了问题能追溯

用ReAct，系统可以展示完整的推理链条。客服答错了？直接看：它问了什么、查了什么、最后怎么答的。开发者定位问题快多了。

传统模型只能两手一摊："抱歉，我不知道为什么会那样。"

### 能随机应变

比如订餐场景：用户说"帮我订个餐厅，要近、评分高、适合商务"。

ReAct会：
1. 获取用户位置。
2. 搜附近高评分餐厅。
3. 筛选适合商务的。
4. 发现首选满了，自动推荐备选。

脚本做不到这种灵活调整——它只会报错或者返回固定结果。

### 能组合多个工具

写一份市场分析报告？ReAct可以协调搜索工具查数据、代码工具做分析、图表工具可视化、写作工具生成报告。模拟的是一个真实分析师的工作方式。

## 工程实践

![工程实践要点](https://mmbiz.qpic.cn/mmbiz_jpg/55IJsnyicJufkaL7MkpwaCsPwqoNpkEoJmlsHpr5icUzDtjIYRRS52lbNPuBkD83dVzYHUDGKXdrfibDXHuKxcQj0RpwawcD7jUicWMB0fiaLrxQ/640?wx_fmt=jpeg&from=appmsg)

### 避免死循环

智能客服常见问题：用户问"客服电话多少"，系统没有这个功能，模型就一直绕圈。

解决办法：

**硬限制**：

```python
max_iterations = 10
for i in range(max_iterations):
    # 执行循环
    if i >= max_iterations:
        return "处理超时，请稍后重试"
```

**循环检测**：同一个操作重复3次，直接判定无法回答，退出。

**系统提示引导**：告诉模型"最多试3次，还是不行就说不知道"。

### 上下文管理

长对话容易爆窗口。比如分析50份简历，每份都查背景信息，上下文直接撑满。

处理方式：

**截断**：只保留最近10轮对话，早期内容丢掉。

**摘要**：把前面的分析压缩成一句话，比如"20份简历分析完成，12份通过初筛"。只留结论，不留过程。

省Token，效果差不多。

### 并行调用

问："对比北京、上海、深圳国庆期间的天气和机票价格。"

串行做法：查北京天气→等，查上海天气→等，查深圳天气→等，查北京机票→等……6次等待。

并行做法：三个城市天气一起查，三张机票一起查。2次等待搞定。

性能差距明显。

### 错误处理

订机票时支付API返回"网络超时"，怎么办？

**分级处理**：
- 网络超时：等1秒重试，最多3次
- API限流：换备用支付渠道
- 余额不足：提示充值
- 系统维护：告知稍后再试

**回退机制**：主渠道失败自动试备用渠道，都失败了保存订单状态，让用户晚点再试。

## 结语

ReAct让AI从被动应答变成主动规划。通过把推理和外部工具结合起来，AI能处理远超传统聊天机器人的复杂任务。

它的核心其实很朴素：**像人一样做事**。想一下、做一下、看结果、接着想。

挑战也不少。循环控制、上下文管理、错误处理，都需要认真对待。但随着技术成熟，基于ReAct的AI智能体正在各个场景里发挥作用。

如果你在搞AI应用，ReAct值得深入了解。
