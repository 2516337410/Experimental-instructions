# Lab 1 System Development Based on LLMs

## 1. Objectives

* Understand concepts related to intelligent model-driven development.

* Master modeling methods assisted by large language models, and use relevant frameworks to build a Multi-Agent Workflow for automated requirements modeling.

## 2. Contents

* **Preparation:** Configure the required environment and select a target model.

* **Task 1:** Design and define the Multi-Agent Workflow, the System Prompt for each Agent, and the Agent output format (DSL).

* **Task 2:** Build the Multi-Agent Workflow to achieve automated domain modeling.

## 3. Materials

* BASE_URL:

* API_KEY:

* Note: <span style="color: red; font-weight: bold;">API_KEY-related information is for this course only and must not be shared with anyone outside this course.</span>

## 4. Assignment Requirements

* Upload the relevant code to GitHub or Gitee. The repository must be set to **Public** visibility, and the **project link** must be submitted in the questionnaire.

* The lab report (provided in the form of the project README file) should include the **input prompts**, **output format definition**, **brief description of the Multi-Agent Workflow**, and **description of the generated requirements model**. No specific format is required.

<!-- * <span style="color: red; font-weight: bold;">Note: The latest submission deadline for this lab is 00:00 on June 23, 2025!</span> -->

## 5. Preparation: Configure the Required Environment and Select a Target Model

* **Target environment setup:** Choose a Multi-Agent Workflow framework, such as [AI SDK](../resource/ai-sdk-tutorial.html), [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/), etc. This document uses ai-sdk as an example. Refer to the following installation steps; for details, read the official documentation:
Step 1 - Create a Next.js project


  ```bash
  npx create-next-app@latest my-ai-app --typescript
  cd my-ai-app
  ```



  Step 2 - Install dependencies


  ```bash
  npm install ai @ai-sdk/openai

  # If using Anthropic Claude:
  npm install ai @ai-sdk/anthropic

  # If using Google Gemini:
  npm install ai @ai-sdk/google
  ```

  Step 3 - Configure the API Key (`.env.local`)
  ```bash
  # OpenAI
  OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx
  ```

* **Select a target model/system:** Refer to [RM2PT Case Study](https://github.com/RM2PT/CaseStudies) ([BUAA Cloud Drive](https://bhpan.buaa.edu.cn/link/AA9A5A244C6E5746F995ED2DBD3082352B)), choose a target system on your own, or extend an existing case.

## 6. Task 1: Design and Define the Agent Output Format (DSL)

* Design the collaborative Multi-Agent process and the corresponding System Prompt for each Agent.

* Design a DSL for Agent output. You may use JSON or another format, but it must include the complete content of the existing requirements model, including use case diagrams, system sequence diagrams, conceptual class diagrams, and OCL contracts.

* Taking a use case diagram as an example, the following DSL can be defined (for reference only):
  
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

## 7. Task 2 Build the Multi-Agent Workflow to Achieve Automated Domain Modeling

* Define and implement the external tools that need to be used.

* Implement the Multi-Agent Workflow based on a relevant framework. This document uses ai-sdk agents as an example, followed by a brief usage guide. For details, read the official documentation: [AI SDK Agents](../resource/ai-sdk-agent-tutorial.html), [AI SDK Advanced](../resource/ai-sdk-advanced-tutorial.html). **You may use any other framework or method for implementation.**

* The output format only needs to follow the **DSL defined in Task 1**.

# 8 Related APIs

## 8.1 AI SDK

### 8.1.1 Multi-turn conversation (`messages` format)

  ```bash
  const { text } = await generateText({
    model: openai('gpt-4o-mini'),
    messages: [
      { role: 'user', content: 'My name is Xiaoming' },
      { role: 'assistant', content: 'Hello, Xiaoming! How can I help you?' },
      { role: 'user', content: 'Do you still remember my name?' },
    ],
  })

  console.log(text)
  // "Of course, I remember. Your name is Xiaoming!"
  ```

### 8.1.2 Return value destructuring

```bash
const result = await generateText({
  model: openai('gpt-4o-mini'),
  prompt: 'Tell me a dad joke',
})

console.log(result.text)           // Main content
console.log(result.usage)          // { promptTokens, completionTokens, totalTokens }
console.log(result.finishReason)   // 'stop' | 'length' | 'tool-calls'
console.log(result.response)       // Raw response object
```

### 8.1.3 Using it in a Next.js API Route

```bash
// app/api/summarize/route.ts
import { generateText } from 'ai'
import { openai } from '@ai-sdk/openai'
import { NextRequest } from 'next/server'

export async function POST(req: NextRequest) {
  const { content } = await req.json()
  
  const { text } = await generateText({
    model: openai('gpt-4o-mini'),
    system: 'Summarize the user input into 3 sentences',
    prompt: content,
  })
  
  return Response.json({ summary: text })
}
```

### 8.1.4 Tool Use

- Define a tool
```bash
import { generateText, tool } from 'ai'
import { openai } from '@ai-sdk/openai'
import { z } from 'zod' // AI SDK uses zod to define parameter types

const { text } = await generateText({
  model: openai('gpt-4o-mini'),
  prompt: 'What is the weather like in Beijing today?',
  tools: {
    getWeather: tool({
      description: 'Get weather information for a specified city',
      inputSchema: z.object({
        city: z.string().describe('City name'),
        unit: z.enum(['celsius', 'fahrenheit'])
          .default('celsius'),
      }),
      execute: async ({ city, unit }) => {
        // You can actually call a weather API here
        return { temperature: 22, condition: 'Sunny', city }
      },
    }),
  },
})

console.log(text)
// "The weather in Beijing is sunny today, with a temperature of 22 degrees C, which is suitable for going out."
```

- Multiple tools + automatic multi-step invocation
```bash
const result = await generateText({
  model: openai('gpt-4o'),
  prompt: 'Check the weather in Beijing and Shanghai, then compare which city is hotter',
  tools: {
    getWeather: tool({/* ... */}),
    compareWeather: tool({/* ... */}),
  },
  maxSteps: 5, // AI can automatically call tools multiple times in sequence
})

// View the AI's reasoning steps
for (const step of result.steps) {
  console.log('Tool calls:', step.toolCalls)
  console.log('Tool results:', step.toolResults)
}
```

- Control tool usage strategy
```bash
generateText({
  model: openai('gpt-4o-mini'),
  tools: { /* ... */ },
  
  // auto: AI decides whether to use tools on its own (default)
  toolChoice: 'auto',
  
  // required: force the AI to use a tool
  toolChoice: 'required',
  
  // none: forbid tool use
  toolChoice: 'none',
  
  // specify that a particular tool must be used
  toolChoice: { type: 'tool', toolName: 'getWeather' },
})
```

### 8.1.5 Structured Output
You need to install Zod first. Zod provides runtime type validation. AI SDK uses it to ensure that the parameter types returned by the AI are correct, and also sends the schema to the model to describe the parameter structure.

```bash
npm install zod
```

- Basic usage
```bash
import 'dotenv/config'
import { generateObject } from 'ai'
import { createOpenAI } from '@ai-sdk/openai'
import { z } from 'zod'

// Read configuration from environment variables
const openai = createOpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  baseURL: process.env.BASE_URL,
})

const modelName = process.env.MODEL

// Define the schema of the returned JSON object (using Zod)
const PersonSchema = z.object({
  name: z.string().describe('Person name'),
  age: z.number().describe('Age'),
  occupation: z.string().describe('Occupation'),
  skills: z.array(z.string()).describe('List of skills'),
  bio: z.string().describe('One-sentence profile'),
  isStudent: z.boolean().describe('Whether the person is a student'),
})

async function main() {
  console.log(`Calling model [${modelName}] to generate a structured JSON object...\n`)

  const { object, usage } = await generateObject({
    model: openai(modelName),
    schema: PersonSchema,
    prompt: 'Generate a fictional profile of a Chinese programmer',
  })

  // Directly obtain a type-safe JSON object without JSON.parse
  console.log('=== Generated JSON Object ===')
  console.log(JSON.stringify(object, null, 2))

  console.log('\n=== Field Access Example ===')
  console.log(`Name: ${object.name}`)
  console.log(`Age: ${object.age}`)
  console.log(`Occupation: ${object.occupation}`)
  console.log(`Skills: ${object.skills.join(', ')}`)
  console.log(`Profile: ${object.bio}`)
  console.log(`Is student: ${object.isStudent}`)
}

main().catch(console.error)
```

- Extract structured data from text
```bash
const email = `
  Sender: Manager Wang
  Time: Tomorrow at 3 PM
  Location: Meeting Room B
  Content: Quarterly review meeting, please prepare the Q3 data report
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
  prompt: `Extract meeting information from the following email:\n${email}`,
})

console.log(meeting.title)       // "Quarterly review meeting"
console.log(meeting.organizer)    // "Manager Wang"
console.log(meeting.preparation)  // ["Prepare the Q3 data report"]
```

- `output` mode selection
```bash
// Mode 1: object (default) - returns a single object
generateObject({ schema: z.object({...}), ... })

// Mode 2: array - returns an array of objects (batch generation)
const { object: products } = await generateObject({
  model: openai('gpt-4o-mini'),
  output: 'array',
  schema: z.object({
    name: z.string(),
    price: z.number(),
  }),
  prompt: 'Generate 5 fictional software products and their prices',
})
// products: [{ name: 'CloudSync Pro', price: 99 }, ...]

// Mode 3: enum - let the AI answer a multiple-choice question
const { object: sentiment } = await generateObject({
  model: openai('gpt-4o-mini'),
  output: 'enum',
  enum: ['positive', 'negative', 'neutral'],
  prompt: 'A new feature was released today, and users responded very positively!',
})
// sentiment: 'positive'
```

- `streamObject` - streaming structured output
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
  prompt: 'List 10 suggestions for learning TypeScript',
})

// As generation proceeds, the partial object is gradually filled
for await (const partial of partialObjectStream) {
  console.log(partial.items?.length) // Gradually increases from 0 to 10
}
```

## 8.2 AI SDK Agents

### 8.2.1 Basic Usage

- Define an Agent
```bash
import { ToolLoopAgent, tool, stepCountIs } from 'ai'
import { openai } from '@ai-sdk/openai'
import { z } from 'zod'

// Note: AI SDK 6 uses the "provider/model" string format
const weatherAgent = new ToolLoopAgent({
  model: "openai/gpt-4o-mini",
  system: 'You are a weather assistant. Answer questions in Chinese',

  tools: {
    getWeather: tool({
      description: 'Get city weather',
      inputSchema: z.object({       // ⚠️ v6 uses inputSchema, not parameters
        city: z.string(),
      }),
      execute: async ({ city }) => ({
        city,
        temp: 22,
        condition: 'Sunny',
      }),
    }),
  },

  stopWhen: stepCountIs(10), // Execute at most 10 steps
})

// Usage: generate (wait for the full result)
const result = await weatherAgent.generate({
  prompt: 'What is the weather like in Beijing today?',
})
console.log(result.text)
console.log(result.steps)   // All execution steps
console.log(result.usage)   // Aggregated token usage
```

- Streaming mode
```bash
// app/api/agent/route.ts
export async function POST(req: Request) {
  const { prompt } = await req.json()

  // stream() returns a ReadableStream
  const result = weatherAgent.stream({ prompt })

  return result.toDataStreamResponse()
}

// The frontend still uses useChat, fully compatible with ordinary streamText
```

- View the Agent's reasoning process
```bash
const result = await agent.generate({ prompt })

for (const step of result.steps) {
  console.log('Step type:', step.stepType)
  // 'initial' | 'tool-result' | 'continue'

  console.log('Tool calls:', step.toolCalls)
  // [{ toolName: 'getWeather', args: { city: 'Beijing' } }]

  console.log('Tool results:', step.toolResults)
  // [{ toolName: 'getWeather', result: { temp: 22 } }]

  console.log('Step text:', step.text)
  console.log('Token usage:', step.usage)
}
```

### 8.2.2 Loop Control
- `stopWhen` - built-in stop conditions
```bash
import {
  ToolLoopAgent,
  stepCountIs,       // stop after executing N steps
  hasToolCall,       // stop after calling a certain tool
  hasNoToolCalls,    // stop when no tool is called (AI considers the task complete)
} from 'ai'

// Example 1: run at most 5 steps (prevent runaway execution)
new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  tools: { ... },
  stopWhen: stepCountIs(5),
})

// Example 2: stop when there are no tool calls (AI considers the task complete)
new ToolLoopAgent({
  stopWhen: hasNoToolCalls(), // This is also the default behavior
})

// Example 3: combined conditions - stop when any condition is met
new ToolLoopAgent({
  stopWhen: [
    stepCountIs(10),     // stop after more than 10 steps
    hasToolCall('done'), // or when the "done" tool is called
  ],
})
```

- Custom `stopWhen` - stop based on business logic
```bash
import { StopCondition } from 'ai'

// stopWhen accepts a function: (stepResult) => boolean
const agent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  tools: { getBalance, transfer, notify },

  stopWhen: (stepResult) => {
    // If the AI calls the "notify" tool, the task is complete
    return stepResult.toolCalls.some(
      c => c.toolName === 'notify'
    )
  },
})
```

- `prepareStep` - insert logic before each step executes
```bash
const agent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  tools: { search, analyze, writeReport },

  prepareStep: async ({ stepNumber, messages, toolCalls }) => {
    // Only allow the search tool in the first 3 steps
    if (stepNumber < 3) {
      return {
        toolChoice: { type: 'tool', toolName: 'search' },
      }
    }

    // If enough search results have been gathered, switch to analysis mode
    const searchCount = toolCalls.filter(c => c.toolName === 'search').length
    if (searchCount >= 3) {
      return {
        // Dynamically inject additional context
        system: 'You have collected enough information. Now write the analysis report',
        toolChoice: { type: 'tool', toolName: 'writeReport' },
      }
    }

    return {} // No changes, execute normally
  },
})
```

- Callbacks - observe each step in real time
```bash
const result = await agent.generate({
  prompt: 'Analyze competitor products',

  // Triggered after each step completes (non-blocking)
  onStep: (step) => {
    console.log(`Step ${step.stepNumber} completed`)
    console.log('Called tools:', step.toolCalls.map(c => c.toolName))

    // Progress can be pushed to the frontend
    progressChannel.send({
      step: step.stepNumber,
      action: step.toolCalls[0]?.toolName,
    })
  },
})
```

### 8.2.3 Memory
- Persist conversation history (short-term memory across sessions)

Load history from the DB before each conversation, and store it back after the conversation ends.
```bash
// lib/memory.ts
import { db } from './db' // Your database client

export async function loadHistory(userId: string) {
  const rows = await db.query(
    `SELECT role, content FROM messages
     WHERE user_id = $1
     ORDER BY created_at DESC
     LIMIT 20`,
    [userId]
  )
  return rows.reverse() // chronological order
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

- Use it in an Agent
```bash
// app/api/chat/route.ts
export async function POST(req: Request) {
  const { userId, message } = await req.json()

  // 1. Load conversation history
  const history = await loadHistory(userId)

  // 2. Run the Agent (with historical context)
  const result = await myAgent.generate({
    messages: [
      ...history,                           // historical messages
      { role: 'user', content: message },  // new message
    ],
  })

  // 3. Store the new message and AI reply
  await saveMessages(userId, [
    { role: 'user', content: message },
    { role: 'assistant', content: result.text },
  ])

  return Response.json({ reply: result.text })
}
```

- User profile memory (long-term personalization)
```bash
// After each conversation: extract and store user information
async function updateUserProfile(
  userId: string,
  conversation: CoreMessage[]
) {
  // Use generateObject to extract user preferences from the conversation
  const { object: profile } = await generateObject({
    model: openai('gpt-4o-mini'),
    schema: z.object({
      name: z.string().optional(),
      preferredLanguage: z.string().optional(),
      interests: z.array(z.string()),
      expertise: z.string().optional(),
    }),
    system: 'Extract user information from the conversation. Only extract explicitly mentioned content',
    messages: conversation,
  })

  await db.upsert('user_profiles', { userId, ...profile })
}

// In the next conversation: inject the profile into system
async function buildSystemWithProfile(userId: string) {
  const profile = await db.find('user_profiles', { userId })
  if (!profile) return 'You are an intelligent assistant'

  return `You are an intelligent assistant.
User information:
- Name: ${profile.name || 'Unknown'}
- Area of expertise: ${profile.expertise || 'Unknown'}
- Interests: ${profile.interests.join(', ')}
Please adjust the depth and style of your answers according to the user's background.`
}
```

- Semantic memory + RAG
```bash
// Store memory: vectorize important information and save it to the database
async function storeMemory(userId: string, text: string) {
  const { embedding } = await embed({
    model: openai.embedding('text-embedding-3-small'),
    value: text,
  })
  await vectorDB.upsert({ userId, text, embedding })
}

// Retrieve memory: find the most relevant historical memories for the current question
async function retrieveMemory(userId: string, query: string) {
  const { embedding } = await embed({
    model: openai.embedding('text-embedding-3-small'),
    value: query,
  })
  return vectorDB.query({ userId, embedding, topK: 5 })
}

// Usage: build an Agent call with relevant memory
const memories = await retrieveMemory(userId, userMessage)
const result = await myAgent.generate({
  system: `You are an intelligent assistant. The following are historical memories related to this conversation:
${memories.map(m => `- ${m.text}`).join('\n')}`,
  prompt: userMessage,
})
```

### 8.2.4 Subagents
- Call Subagents
```bash
// 1. Define expert Agents
const researchAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: 'You are a professional market research expert. Output structured data',
  tools: { webSearch, fetchPage },
})

const writerAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: 'You are a business copywriting expert. Output professional reports',
  tools: { formatDocument },
})

// 2. Wrap child Agents as tools
const researchTool = tool({
  description: 'Conduct in-depth market research on a topic',
  inputSchema: z.object({
    topic: z.string().describe('Research topic'),
  }),
  execute: async ({ topic }) => {
    const result = await researchAgent.generate({
      prompt: `Conduct in-depth research on: ${topic}, and provide data and insights`,
    })
    return result.text  // Return the research result
  },
})

const writeTool = tool({
  description: 'Write a professional report based on research data',
  inputSchema: z.object({
    researchData: z.string(),
    reportType: z.enum(['executive', 'detailed']),
  }),
  execute: async ({ researchData, reportType }) => {
    const result = await writerAgent.generate({
      prompt: `Write a ${reportType} report based on the following data:\n${researchData}`,
    })
    return result.text
  },
})

// 3. Orchestrator Agent - automatically coordinate child Agents
const orchestrator = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: 'You are a project manager who coordinates experts to complete complex tasks',
  tools: { researchTool, writeTool },
  stopWhen: stepCountIs(8),
})

// 4. Usage: one prompt triggers the complete multi-Agent workflow
const result = await orchestrator.generate({
  prompt: 'Research the Chinese new energy vehicle market and generate an executive summary report',
})
// The Orchestrator will automatically:
// 1. Call researchTool (which triggers researchAgent)
// 2. Use the research result to call writeTool (which triggers writerAgent)
// 3. Merge the results and provide the final answer
```

- Execute Subagents in parallel
```bash
// Multiple child Agents run simultaneously, waited on with Promise.all
async function parallelAnalysis(company: string) {
  const [finance, market, tech] = await Promise.all([
    financeAgent.generate({
      prompt: `Analyze the financial condition of ${company}`,
    }),
    marketAgent.generate({
      prompt: `Analyze the market position of ${company}`,
    }),
    techAgent.generate({
      prompt: `Evaluate the technical strength of ${company}`,
    }),
  ])

  // Use the Orchestrator to merge the analysis from three dimensions
  const summary = await orchestrator.generate({
    prompt: `Synthesize the following analyses from three dimensions and provide investment advice:
Financial: ${finance.text}
Market: ${market.text}
Technology: ${tech.text}`,
  })

  return summary.text
}
```

- Dynamic routing Agent - dispatch based on the question
```bash
// Expert Agents for different domains
const agents = {
  code:    new ToolLoopAgent({ system: 'You are a code expert', ... }),
  legal:   new ToolLoopAgent({ system: 'You are a legal advisor', ... }),
  finance: new ToolLoopAgent({ system: 'You are a financial analyst', ... }),
}

async function route(userQuestion: string) {
  // First use a lightweight model for classification
  const { object: { domain } } = await generateObject({
    model: openai('gpt-4o-mini'),
    schema: z.object({
      domain: z.enum(['code', 'legal', 'finance', 'general']),
    }),
    prompt: `Which domain does this question belong to: ${userQuestion}`,
  })

  // Route to the corresponding expert
  const agent = agents[domain] ?? defaultAgent
  const result = await agent.generate({ prompt: userQuestion })
  return result.text
}
```

### Multi-Agent
AI SDK itself does not include a built-in "Multi-Agent framework", but it provides all the primitives needed to build multi-Agent collaboration. Use `ToolLoopAgent` + asynchronous communication patterns to implement a Multi-Agent network.

- Pattern 1: Parallel network

Multiple expert Agents run simultaneously, and the results are consolidated by a Synthesizer Agent.
```bash
import { ToolLoopAgent, tool, stepCountIs } from 'ai'

// -- Define the expert Agent network --
const agents = {
  frontend: new ToolLoopAgent({
    model: 'openai/gpt-4o',
    system: 'You are a frontend architecture expert, focusing on React/TypeScript/performance optimization',
    tools: { searchDocs, runLinter },
  }),

  backend: new ToolLoopAgent({
    model: 'openai/gpt-4o',
    system: 'You are a backend architecture expert, focusing on API design/database/security',
    tools: { searchDocs, queryDB },
  }),

  security: new ToolLoopAgent({
    model: 'anthropic/claude-sonnet-4-5', // Different models can be mixed!
    system: 'You are a security expert, focusing on vulnerability identification/OWASP/penetration testing',
    tools: { scanVulnerabilities, checkDependencies },
  }),

  synthesizer: new ToolLoopAgent({
    model: 'openai/gpt-4o',
    system: 'You are a technical director responsible for integrating multiple expert opinions and giving the final solution',
  }),
}

// -- Run three experts in parallel, then synthesize --
async function codeReview(prDescription: string) {
  // Step 1: three Agents analyze simultaneously (saving 2/3 of the time)
  const [frontendReview, backendReview, securityReview] =
    await Promise.all([
      agents.frontend.generate({
        prompt: `Review the frontend part of this PR:\n${prDescription}`,
      }),
      agents.backend.generate({
        prompt: `Review the backend part of this PR:\n${prDescription}`,
      }),
      agents.security.generate({
        prompt: `Check the security risks of this PR:\n${prDescription}`,
      }),
    ])

  // Step 2: the Synthesizer integrates all opinions
  const finalReview = await agents.synthesizer.generate({
    prompt: `Integrate the following Code Review opinions from three experts, rank them by priority, and give final recommendations:

【Frontend Expert】
${frontendReview.text}

【Backend Expert】
${backendReview.text}

【Security Expert】
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

- Pattern 2: Event-driven network
```bash
import { EventEmitter } from 'events'

// Use EventEmitter as the message bus between Agents
const bus = new EventEmitter()

// Agent A: Requirements analyst - analyze user requirements and produce technical specifications
const analystAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  system: 'You are a requirements analyst who converts user requirements into a technical specification document',
})

// Agent B: Developer - generate code after receiving the specification
const devAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: 'You are a senior developer who implements code according to the technical specification',
  tools: { writeFile, runTests },
})

// Agent C: QA - run tests and report issues after receiving the code
const qaAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  system: 'You are a QA engineer who reviews code and reports issues',
  tools: { runTests, checkCoverage },
})

// -- Event subscriptions (communication protocol between Agents) --
bus.on('spec-ready', async (spec: string) => {
  console.log('[Dev Agent] Specification received, start coding...')
  const result = await devAgent.generate({
    prompt: `Implement code according to the following specification:\n${spec}`,
  })
  bus.emit('code-ready', result.text) // Trigger the next Agent
})

bus.on('code-ready', async (code: string) => {
  console.log('[QA Agent] Code received, start testing...')
  const result = await qaAgent.generate({
    prompt: `Review the following code and report issues:\n${code}`,
  })
  bus.emit('review-done', result.text)
})

// -- Start the pipeline --
async function runPipeline(userRequirement: string) {
  const spec = await analystAgent.generate({ prompt: userRequirement })
  bus.emit('spec-ready', spec.text) // Start the whole chain
}

await runPipeline('Implement a user login API with JWT authentication support')
```

- Pattern 3: Shared state
```bash
// Shared state object (in a real project, put it in Redis/DB)
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

// Tools: Agents read and write shared state through tools
const stateTools = {
  readState: tool({
    description: 'Read the current project state',
    inputSchema: z.object({}),
    execute: async () => projectState,
  }),
  updateState: tool({
    description: 'Update a field in the project state',
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

// Each Agent mounts stateTools and can read/write the shared state
const pmAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o-mini',
  system: 'You are a PM responsible for organizing requirements and writing them into the project state',
  tools: { ...stateTools },
})

const architectAgent = new ToolLoopAgent({
  model: 'openai/gpt-4o',
  system: 'You are an architect who reads requirements, formulates a technical design, and writes it into the state',
  tools: { ...stateTools },
})

// Sequential collaboration: PM organizes requirements first, then the architect reads them and designs the solution
await pmAgent.generate({ prompt: 'Organize the user-provided requirements' })
await architectAgent.generate({ prompt: 'Read the requirements and output the technical solution' })
```
