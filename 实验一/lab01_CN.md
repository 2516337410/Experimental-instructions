# 实验一 基于 LLM 的系统开发

## 1. 实验目标

* 了解智能模型驱动相关概念

* 掌握基于大语言模型辅助的建模方法，基于相关框架构建MultiAgent Workflow实现自动化需求建模。

## 2. 实验内容

* **实验准备：** 相关环境配置、选定目标模型

* **任务1：** 设计和定义MultiAgent Workflow、Agent对应的System Prompt以及Agent输出格式（DSL）。

* **任务2：** 构建MultiAgent Workflow，实现自动化领域建模。

## 3. 实验材料

* BASE_URL：

* API_KEY：

* 注意：<span style="color: red; font-weight: bold;">API_KEY相关信息仅限本课程使用，不要提供给本课程之外的人员。</span>

## 4. 作业要求

* 相关代码上传到Github或Gitee中，仓库需要设置为**Public**可见，并将**项目链接**填写到问卷中。

* 实验报告（以项目README文件形式给出），包含**输入的Prompt**，**输出格式定义**，**MultiAgent Workflow简要说明**，**生成的需求模型说明**，不要求具体格式。

<!-- * <span style="color: red; font-weight: bold;">注意：本次实验最晚提交时间为 2025 年 6 月 23 日 0 点！</span> -->

## 5. 实验准备：相关环境配置、选定目标模型

* **目标环境配置：** 选择MultiAgent Workflow框架，如[AI SDK](../resource/ai-sdk-tutorial.html)、[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)等。本文档以ai-sdk为例，安装方式参考如下，具体可阅读官方文档：
Step 1 — 创建 Next.js 项目


  ```bash
  npx create-next-app@latest my-ai-app --typescript
  cd my-ai-app
  ```



  Step 2 — 安装依赖


  ```bash
  npm install ai @ai-sdk/openai

  # 如果用 Anthropic Claude：
  npm install ai @ai-sdk/anthropic

  # 如果用 Google Gemini：
  npm install ai @ai-sdk/google
  ```

  Step 3 — 配置 API Key（.env.local）
  ```bash
  # OpenAI
  OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx
  ```

* **选定目标模型：** 参考[RM2PT Case Study](https://github.com/RM2PT/CaseStudies)（[北航云盘](https://bhpan.buaa.edu.cn/link/AA9A5A244C6E5746F995ED2DBD3082352B)）, 自选目标系统或者在已有案例基础上进行扩展。

## 6. 任务1：设计和定义Agent输出格式（DSL）

* 设计MultiAgent协同的流程、Agent对应的System Prompt。

* 设计用于Agent输出的DSL，可以使用json或其他形式，需要包含现有需求模型完整内容，包括用例图、系统顺序图、概念类图以及OCL合约。

* 以用例图为例，可以定义如下的DSL（仅供参考）
  
  ```json
  {
      "name": "CoCoME",
      "usecases": [
          {
              "name": "OpenStore",
              "includes": [],
              "extends": []
          }
          {
              "name": "CloseStore",
              "includes": [],
              "extends": []
          }
          {
              "name": "manageStore"
              "includes": ["OpenStore", "CloseStore"],
              "extends": []
          }
      ]
  }
  ```

## 7. 任务2 构建MultiAgent Workflow，实现自动化领域建模

* 定义和实现需要使用的外部tool。

* 基于相关框架实现MultiAgent Workflow，本文档以ai-sdk agents为例，后附简要使用说明，详细信息可阅读官方文档：[AI SDK Agents](../resource/ai-sdk-agent-tutorial.html)、[AI SDK Advanced](../resource/ai-sdk-advanced-tutorial.html)。**可以使用其他任意框架或方法实现。**

* 输出格式以**任务1中定义的DSL**即可。

# 8 相关接口

## 8.1 AI SDK

### 8.1.1 多轮对话（messages 格式）

  ```bash
  const { text } = await generateText({
    model: openai('gpt-4o-mini'),
    messages: [
      { role: 'user', content: '我叫小明' },
      { role: 'assistant', content: '你好，小明！有什么我能帮你的？' },
      { role: 'user', content: '你还记得我叫什么吗？' },
    ],
  })

  console.log(text)
  // "当然记得，你叫小明！"
  ```

### 8.1.2 返回值解构

```bash
const result = await generateText({
  model: openai('gpt-4o-mini'),
  prompt: '讲个冷笑话',
})

console.log(result.text)           // 主要内容
console.log(result.usage)          // { promptTokens, completionTokens, totalTokens }
console.log(result.finishReason)   // 'stop' | 'length' | 'tool-calls'
console.log(result.response)       // 原始响应对象
```

### 8.1.3 在 Next.js API Route 中使用

```bash
// app/api/summarize/route.ts
import { generateText } from 'ai'
import { openai } from '@ai-sdk/openai'
import { NextRequest } from 'next/server'

export async function POST(req: NextRequest) {
  const { content } = await req.json()
  
  const { text } = await generateText({
    model: openai('gpt-4o-mini'),
    system: '将用户输入摘要为3句话',
    prompt: content,
  })
  
  return Response.json({ summary: text })
}
```

### 8.1.4 Tool Use

- 定义工具
```bash
import { generateText, tool } from 'ai'
import { openai } from '@ai-sdk/openai'
import { z } from 'zod' // AI SDK 用 zod 定义参数类型

const { text } = await generateText({
  model: openai('gpt-4o-mini'),
  prompt: '北京今天天气怎么样？',
  tools: {
    getWeather: tool({
      description: '获取指定城市的天气信息',
      inputSchema: z.object({
        city: z.string().describe('城市名称'),
        unit: z.enum(['celsius', 'fahrenheit'])
          .default('celsius'),
      }),
      execute: async ({ city, unit }) => {
        // 这里可以真的调用天气 API
        return { temperature: 22, condition: '晴', city }
      },
    }),
  },
})

console.log(text)
// "北京今天天气晴朗，气温 22°C，适合外出。"
```

- 多工具 + 自动多步调用
```bash
const result = await generateText({
  model: openai('gpt-4o'),
  prompt: '查询北京和上海的天气，然后比较哪个更热',
  tools: {
    getWeather: tool({/* ... */}),
    compareWeather: tool({/* ... */}),
  },
  maxSteps: 5, // AI 可以自动连续调用多次工具
})

// 查看 AI 的推理步骤
for (const step of result.steps) {
  console.log('工具调用:', step.toolCalls)
  console.log('工具结果:', step.toolResults)
}
```

- 控制工具使用策略
```bash
generateText({
  model: openai('gpt-4o-mini'),
  tools: { /* ... */ },
  
  // auto: AI 自己决定要不要用工具（默认）
  toolChoice: 'auto',
  
  // required: 强制 AI 必须用工具
  toolChoice: 'required',
  
  // none: 禁止用工具
  toolChoice: 'none',
  
  // 指定必须用某个特定工具
  toolChoice: { type: 'tool', toolName: 'getWeather' },
})
```

### 8.1.5 结构化输出
需要先安装Zod。Zod 提供了运行时类型校验，AI SDK 用它来确保 AI 传来的参数类型正确，同时也会把 schema 发给模型告知参数结构。

```bash
npm install zod
```

- 基础用法
```bash
import 'dotenv/config'
import { generateObject } from 'ai'
import { createOpenAI } from '@ai-sdk/openai'
import { z } from 'zod'

// 从环境变量读取配置
const openai = createOpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  baseURL: process.env.BASE_URL,
})

const modelName = process.env.MODEL

// 定义返回的 JSON 对象的 schema（使用 Zod）
const PersonSchema = z.object({
  name: z.string().describe('人物姓名'),
  age: z.number().describe('年龄'),
  occupation: z.string().describe('职业'),
  skills: z.array(z.string()).describe('技能列表'),
  bio: z.string().describe('一句话简介'),
  isStudent: z.boolean().describe('是否是学生'),
})

async function main() {
  console.log(`正在调用模型 [${modelName}]，生成结构化 JSON 对象...\n`)

  const { object, usage } = await generateObject({
    model: openai(modelName),
    schema: PersonSchema,
    prompt: '生成一个虚构的中国程序员角色信息',
  })

  // 直接拿到类型安全的 JSON 对象，无需 JSON.parse
  console.log('=== 生成的 JSON 对象 ===')
  console.log(JSON.stringify(object, null, 2))

  console.log('\n=== 字段访问示例 ===')
  console.log(`姓名: ${object.name}`)
  console.log(`年龄: ${object.age}`)
  console.log(`职业: ${object.occupation}`)
  console.log(`技能: ${object.skills.join(', ')}`)
  console.log(`简介: ${object.bio}`)
  console.log(`是否学生: ${object.isStudent}`)
}

main().catch(console.error)
```

- 从文本提取结构化数据
```bash
const email = `
  发件人: 王经理
  时间: 明天下午3点
  地点: 会议室B
  内容: 季度复盘会议，请准备Q3数据报告
`

const { object: meeting } = await generateObject({
  model: openai('gpt-4o-mini'),
  schema: z.object({
    title: z.string(),
    organizer: z.string(),
    location: z.string(),
    time: z.string(),
    preparation: z.array(z.string()),
  }),
  prompt: `从以下邮件中提取会议信息：\n${email}`,
})

console.log(meeting.title)       // "季度复盘会议"
console.log(meeting.organizer)    // "王经理"
console.log(meeting.preparation)  // ["准备Q3数据报告"]
```

- output 模式选择
```bash
// 模式1: object（默认）—— 返回单个对象
generateObject({ schema: z.object({...}), ... })

// 模式2: array —— 返回对象数组（批量生成）
const { object: products } = await generateObject({
  model: openai('gpt-4o-mini'),
  output: 'array',
  schema: z.object({
    name: z.string(),
    price: z.number(),
  }),
  prompt: '生成5个虚构的软件产品及其价格',
})
// products: [{ name: 'CloudSync Pro', price: 99 }, ...]

// 模式3: enum —— 让 AI 做选择题
const { object: sentiment } = await generateObject({
  model: openai('gpt-4o-mini'),
  output: 'enum',
  enum: ['positive', 'negative', 'neutral'],
  prompt: '今天发布新功能，用户反馈很好！',
})
// sentiment: 'positive'
```

- streamObject — 流式结构化输出
```bash
import { streamObject } from 'ai'

const { partialObjectStream } = streamObject({
  model: openai('gpt-4o-mini'),
  schema: z.object({
    items: z.array(z.object({
      title: z.string(),
      desc: z.string(),
    }))
  }),
  prompt: '列出10个学习 TypeScript 的建议',
})

// 随着生成进行，partial 对象逐渐填充
for await (const partial of partialObjectStream) {
  console.log(partial.items?.length) // 逐渐从 0 增加到 10
}
```

## 8.2 AI SDK Agents

### 8.2.1 基本用法

- 定义 Agent
```bash
import { ToolLoopAgent, tool, stepCountIs } from 'ai'
import { openai } from '@ai-sdk/openai'
import { z } from 'zod'

// 注意：AI SDK 6 用 "provider/model" 字符串格式
const weatherAgent = new ToolLoopAgent({
  model: "openai/gpt-4o-mini",
  system: '你是天气助手，用中文回答问题',

  tools: {
    getWeather: tool({
      description: '获取城市天气',
      inputSchema: z.object({       // ⚠️ v6 用 inputSchema，不是 parameters
        city: z.string(),
      }),
      execute: async ({ city }) => ({
        city,
        temp: 22,
        condition: '晴',
      }),
    }),
  },

  stopWhen: stepCountIs(10), // 最多执行 10 步
})

// 使用：generate（等待完整结果）
const result = await weatherAgent.generate({
  prompt: '北京今天天气怎么样？',
})
console.log(result.text)
console.log(result.steps)   // 所有执行步骤
console.log(result.usage)   // token 用量汇总
```

- 流式模式
```bash
// app/api/agent/route.ts
export async function POST(req: Request) {
  const { prompt } = await req.json()

  // stream() 返回 ReadableStream
  const result = weatherAgent.stream({ prompt })

  return result.toDataStreamResponse()
}

// 前端仍然用 useChat，与普通 streamText 完全兼容
```

- 查看 Agent 的推理过程
```bash
const result = await agent.generate({ prompt })

for (const step of result.steps) {
  console.log('步骤类型:', step.stepType)
  // 'initial' | 'tool-result' | 'continue'

  console.log('工具调用:', step.toolCalls)
  // [{ toolName: 'getWeather', args: { city: '北京' } }]

  console.log('工具结果:', step.toolResults)
  // [{ toolName: 'getWeather', result: { temp: 22 } }]

  console.log('本步骤文本:', step.text)
  console.log('Token用量:', step.usage)
}
```

### 8.2.2 循环控制
- stopWhen — 内置终止条件
```bash
import {
  ToolLoopAgent,
  stepCountIs,       // 执行了 N 步就停
  hasToolCall,       // 调用了某个工具就停
  hasNoToolCalls,    // 没有调用工具就停（AI 认为完成）
} from 'ai'

// 示例 1：最多跑 5 步（防止失控）
new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  tools: { ... },
  stopWhen: stepCountIs(5),
})

// 示例 2：没有工具调用时停止（AI 认为任务完成）
new ToolLoopAgent({
  stopWhen: hasNoToolCalls(), // 这也是默认行为
})

// 示例 3：组合条件 — 满足任一即停
new ToolLoopAgent({
  stopWhen: [
    stepCountIs(10),     // 超过 10 步停止
    hasToolCall('done'), // 或调用了"完成"工具
  ],
})
```

- 自定义 stopWhen — 根据业务逻辑停止
```bash
import { StopCondition } from 'ai'

// stopWhen 接收一个函数：(stepResult) => boolean
const agent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  tools: { getBalance, transfer, notify },

  stopWhen: (stepResult) => {
    // 如果 AI 调用了"发送通知"工具，说明已完成
    return stepResult.toolCalls.some(
      c => c.toolName === 'notify'
    )
  },
})
```

- prepareStep — 每步执行前插入逻辑
```bash
const agent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  tools: { search, analyze, writeReport },

  prepareStep: async ({ stepNumber, messages, toolCalls }) => {
    // 前 3 步只能用 search 工具
    if (stepNumber < 3) {
      return {
        toolChoice: { type: 'tool', toolName: 'search' },
      }
    }

    // 如果有足够多的搜索结果，切换到分析模式
    const searchCount = toolCalls.filter(c => c.toolName === 'search').length
    if (searchCount >= 3) {
      return {
        // 动态注入额外上下文
        system: '你已收集了足够信息，现在请撰写分析报告',
        toolChoice: { type: 'tool', toolName: 'writeReport' },
      }
    }

    return {} // 不修改，正常执行
  },
})
```

- 回调 — 实时观察每步结果
```bash
const result = await agent.generate({
  prompt: '分析竞争对手产品',

  // 每步完成后触发（不阻塞执行）
  onStep: (step) => {
    console.log(`第 ${step.stepNumber} 步完成`)
    console.log('调用工具:', step.toolCalls.map(c => c.toolName))

    // 可以把进度推送给前端
    progressChannel.send({
      step: step.stepNumber,
      action: step.toolCalls[0]?.toolName,
    })
  },
})
```

### 8.2.3 Memory
- 持久化对话历史（跨会话短期记忆）

每次对话前从 DB 加载历史，对话结束后存回 DB。
```bash
// lib/memory.ts
import { db } from './db' // 你的数据库客户端

export async function loadHistory(userId: string) {
  const rows = await db.query(
    `SELECT role, content FROM messages
     WHERE user_id = $1
     ORDER BY created_at DESC
     LIMIT 20`,
    [userId]
  )
  return rows.reverse() // 按时间正序
}

export async function saveMessages(
  userId: string,
  messages: CoreMessage[]
) {
  await db.query(
    `INSERT INTO messages (user_id, role, content)
     VALUES ($1, $2, $3)`,
    messages.map(m => [userId, m.role, m.content])
  )
}
```

- 在 Agent 中使用
```bash
// app/api/chat/route.ts
export async function POST(req: Request) {
  const { userId, message } = await req.json()

  // 1. 加载历史对话
  const history = await loadHistory(userId)

  // 2. 运行 Agent（带历史上下文）
  const result = await myAgent.generate({
    messages: [
      ...history,                           // 历史消息
      { role: 'user', content: message },  // 新消息
    ],
  })

  // 3. 存储新消息和 AI 回复
  await saveMessages(userId, [
    { role: 'user', content: message },
    { role: 'assistant', content: result.text },
  ])

  return Response.json({ reply: result.text })
}
```

- 用户画像记忆（长期个性化）
```bash
// 每次对话结束后：提取并存储用户信息
async function updateUserProfile(
  userId: string,
  conversation: CoreMessage[]
) {
  // 用 generateObject 从对话中提取用户偏好
  const { object: profile } = await generateObject({
    model: openai('gpt-4o-mini'),
    schema: z.object({
      name: z.string().optional(),
      preferredLanguage: z.string().optional(),
      interests: z.array(z.string()),
      expertise: z.string().optional(),
    }),
    system: '从对话中提取用户信息，只提取明确提及的内容',
    messages: conversation,
  })

  await db.upsert('user_profiles', { userId, ...profile })
}

// 下次对话时：把画像注入 system
async function buildSystemWithProfile(userId: string) {
  const profile = await db.find('user_profiles', { userId })
  if (!profile) return '你是一个智能助手'

  return `你是一个智能助手。
用户信息：
- 姓名：${profile.name || '未知'}
- 擅长领域：${profile.expertise || '未知'}
- 兴趣：${profile.interests.join(', ')}
请根据用户背景调整回答的深度和风格。`
}
```

- 语义记忆 + RAG
```bash
// 存储记忆：把重要信息向量化存入数据库
async function storeMemory(userId: string, text: string) {
  const { embedding } = await embed({
    model: openai.embedding('text-embedding-3-small'),
    value: text,
  })
  await vectorDB.upsert({ userId, text, embedding })
}

// 检索记忆：找出与当前问题最相关的历史记忆
async function retrieveMemory(userId: string, query: string) {
  const { embedding } = await embed({
    model: openai.embedding('text-embedding-3-small'),
    value: query,
  })
  return vectorDB.query({ userId, embedding, topK: 5 })
}

// 使用：构建带相关记忆的 Agent 调用
const memories = await retrieveMemory(userId, userMessage)
const result = await myAgent.generate({
  system: `你是智能助手。以下是与此次对话相关的历史记忆：
${memories.map(m => `- ${m.text}`).join('\n')}`,
  prompt: userMessage,
})
```

### 8.2.4 Subagents
- 调用 Subagents
```bash
// 1. 定义专家 Agent
const researchAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: '你是专业的市场调研专家，输出结构化数据',
  tools: { webSearch, fetchPage },
})

const writerAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: '你是商业文案撰写专家，输出专业报告',
  tools: { formatDocument },
})

// 2. 把子 Agent 包装成工具
const researchTool = tool({
  description: '对某个主题进行深度市场调研',
  inputSchema: z.object({
    topic: z.string().describe('调研主题'),
  }),
  execute: async ({ topic }) => {
    const result = await researchAgent.generate({
      prompt: `深度调研：${topic}，提供数据和洞察`,
    })
    return result.text  // 返回调研结果
  },
})

const writeTool = tool({
  description: '根据调研数据撰写专业报告',
  inputSchema: z.object({
    researchData: z.string(),
    reportType: z.enum(['executive', 'detailed']),
  }),
  execute: async ({ researchData, reportType }) => {
    const result = await writerAgent.generate({
      prompt: `基于以下数据写${reportType}报告：\n${researchData}`,
    })
    return result.text
  },
})

// 3. Orchestrator Agent — 自动协调子 Agent
const orchestrator = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: '你是项目经理，协调专家完成复杂任务',
  tools: { researchTool, writeTool },
  stopWhen: stepCountIs(8),
})

// 4. 使用：一个 prompt 触发完整的多 Agent 流程
const result = await orchestrator.generate({
  prompt: '调研中国新能源汽车市场，并生成一份管理层摘要报告',
})
// Orchestrator 会自动：
// 1. 调用 researchTool（触发 researchAgent）
// 2. 用调研结果调用 writeTool（触发 writerAgent）
// 3. 合并结果给出最终答案
```

- 并行执行 Subagents
```bash
// 多个子 Agent 同时运行，用 Promise.all 等待
async function parallelAnalysis(company: string) {
  const [finance, market, tech] = await Promise.all([
    financeAgent.generate({
      prompt: `分析 ${company} 的财务状况`,
    }),
    marketAgent.generate({
      prompt: `分析 ${company} 的市场地位`,
    }),
    techAgent.generate({
      prompt: `评估 ${company} 的技术实力`,
    }),
  ])

  // 用 Orchestrator 合并三个维度的分析
  const summary = await orchestrator.generate({
    prompt: `综合以下三个维度的分析，给出投资建议：
财务：${finance.text}
市场：${market.text}
技术：${tech.text}`,
  })

  return summary.text
}
```

- 动态路由 Agent — 根据问题分流
```bash
// 不同领域的专家 Agent
const agents = {
  code:    new ToolLoopAgent({ system: '你是代码专家', ... }),
  legal:   new ToolLoopAgent({ system: '你是法律顾问', ... }),
  finance: new ToolLoopAgent({ system: '你是财务分析师', ... }),
}

async function route(userQuestion: string) {
  // 先用轻量模型做分类
  const { object: { domain } } = await generateObject({
    model: openai('gpt-4o-mini'),
    schema: z.object({
      domain: z.enum(['code', 'legal', 'finance', 'general']),
    }),
    prompt: `这个问题属于哪个领域：${userQuestion}`,
  })

  // 路由到对应专家
  const agent = agents[domain] ?? defaultAgent
  const result = await agent.generate({ prompt: userQuestion })
  return result.text
}
```

### Multi-Agent
AI SDK 本身不内置"Multi-Agent 框架"，但它提供了构建多 Agent 协作所需的全部原语。用 ToolLoopAgent + 异步通信模式来实现 Multi-Agent 网络。

- 模式 1：并行网络

多个专家 Agent 同时运行，结果汇总给 Synthesizer Agent 整合。
```bash
import { ToolLoopAgent, tool, stepCountIs } from 'ai'

// ── 定义专家 Agent 网络 ──
const agents = {
  frontend: new ToolLoopAgent({
    model: 'openai/gpt-4o',
    system: '你是前端架构专家，专注 React/TypeScript/性能优化',
    tools: { searchDocs, runLinter },
  }),

  backend: new ToolLoopAgent({
    model: 'openai/gpt-4o',
    system: '你是后端架构专家，专注 API 设计/数据库/安全',
    tools: { searchDocs, queryDB },
  }),

  security: new ToolLoopAgent({
    model: 'anthropic/claude-sonnet-4-5', // 可以混用不同模型！
    system: '你是安全专家，专注漏洞识别/OWASP/渗透测试',
    tools: { scanVulnerabilities, checkDependencies },
  }),

  synthesizer: new ToolLoopAgent({
    model: 'openai/gpt-4o',
    system: '你是技术总监，负责整合多个专家意见给出最终方案',
  }),
}

// ── 并行执行三个专家，再汇总 ──
async function codeReview(prDescription: string) {
  // Step 1: 三个 Agent 同时分析（节省 2/3 时间）
  const [frontendReview, backendReview, securityReview] =
    await Promise.all([
      agents.frontend.generate({
        prompt: `审查这个 PR 的前端部分：\n${prDescription}`,
      }),
      agents.backend.generate({
        prompt: `审查这个 PR 的后端部分：\n${prDescription}`,
      }),
      agents.security.generate({
        prompt: `检查这个 PR 的安全风险：\n${prDescription}`,
      }),
    ])

  // Step 2: Synthesizer 整合所有意见
  const finalReview = await agents.synthesizer.generate({
    prompt: `整合以下三位专家的 Code Review 意见，给出优先级排序和最终建议：

【前端专家】
${frontendReview.text}

【后端专家】
${backendReview.text}

【安全专家】
${securityReview.text}`,
  })

  return {
    final: finalReview.text,
    details: { frontendReview, backendReview, securityReview },
    totalSteps: frontendReview.steps.length
      + backendReview.steps.length
      + securityReview.steps.length,
  }
}
```

- 模式 2：事件驱动网络
```bash
import { EventEmitter } from 'events'

// 用 EventEmitter 作为 Agent 间的消息总线
const bus = new EventEmitter()

// Agent A：需求分析师——分析用户需求，产出技术规格
const analystAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  system: '你是需求分析师，将用户需求转化为技术规格文档',
})

// Agent B：开发者——收到规格后生成代码
const devAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: '你是高级开发者，根据技术规格实现代码',
  tools: { writeFile, runTests },
})

// Agent C：QA——收到代码后执行测试并报告
const qaAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  system: '你是 QA 工程师，审查代码并报告问题',
  tools: { runTests, checkCoverage },
})

// ── 事件订阅（Agent 之间的通信协议）──
bus.on('spec-ready', async (spec: string) => {
  console.log('[Dev Agent] 收到规格，开始编码...')
  const result = await devAgent.generate({
    prompt: `根据以下规格实现代码：\n${spec}`,
  })
  bus.emit('code-ready', result.text) // 触发下一个 Agent
})

bus.on('code-ready', async (code: string) => {
  console.log('[QA Agent] 收到代码，开始测试...')
  const result = await qaAgent.generate({
    prompt: `审查以下代码并报告问题：\n${code}`,
  })
  bus.emit('review-done', result.text)
})

// ── 启动流水线 ──
async function runPipeline(userRequirement: string) {
  const spec = await analystAgent.generate({ prompt: userRequirement })
  bus.emit('spec-ready', spec.text) // 启动整个链路
}

await runPipeline('实现一个用户登录 API，支持 JWT 认证')
```

- 模式 3：共享状态
```bash
// 共享状态对象（实际项目中放在 Redis/DB）
interface ProjectState {
  requirements: string[]
  designDecisions: string[]
  implementedFeatures: string[]
  openIssues: string[]
  status: 'planning' | 'building' | 'reviewing' | 'done'
}

const projectState: ProjectState = {
  requirements: [],
  designDecisions: [],
  implementedFeatures: [],
  openIssues: [],
  status: 'planning',
}

// 工具：Agent 通过工具读写共享状态
const stateTools = {
  readState: tool({
    description: '读取当前项目状态',
    inputSchema: z.object({}),
    execute: async () => projectState,
  }),
  updateState: tool({
    description: '更新项目状态中的某个字段',
    inputSchema: z.object({
      field: z.enum(['requirements', 'designDecisions',
                        'implementedFeatures', 'openIssues']),
      items: z.array(z.string()),
    }),
    execute: async ({ field, items }) => {
      projectState[field].push(...items)
      return { success: true, updated: field }
    },
  }),
}

// 每个 Agent 都挂载 stateTools，可以读写共享状态
const pmAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  system: '你是 PM，负责整理需求并写入项目状态',
  tools: { ...stateTools },
})

const architectAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: '你是架构师，读取需求，制定技术设计方案并写入状态',
  tools: { ...stateTools },
})

// 顺序协作：PM 先整理需求，架构师再读取设计
await pmAgent.generate({ prompt: '整理用户提出的需求' })
await architectAgent.generate({ prompt: '读取需求，输出技术方案' })
```
