

> Originally published on Medium via **OpusClip Engineering**; reposted here on my blog.  
> Original: [Bridging the Gap: How Engineers and PMs Ship Winning LLM Features Together](https://medium.com/opus-engineering/bridging-the-gap-how-engineers-and-pms-ship-winning-llm-features-together-147a36ab4089)

# How Engineers and PMs Ship Winning LLM Features Faster: 3 Technical Decisions

**TL;DR:**

- **Prompts belong in configs, not code:** Enable rapid iteration without deployments
- **Variables go last:** Save 90% on costs through KV-cache optimization
- **Separate semantics from schema**: PMs own meaning, engineers own structure

The best LLM features aren’t built in silos. When engineers and PMs at **OpusClip** started collaborating on prompt architecture, **iteration cycles dropped from days to minutes**, **API costs fell by 10x**, and **prompts in production are more reliable**. Here are the three technical decisions that made the biggest difference.

### PM to Prompt Distance

**What it is:** The number of hops — and the amount of interpretation — between your product requirement and the exact text/settings the model actually receives.

![](https://cdn-images-1.medium.com/max/1024/1*zLLFbHw29DqVCOuTB7jkcQ.png)

[Tao Zhang first defines this metric in Manus.](https://www.linkedin.com/posts/manus-im_product-to-prompt-distance-is-fast-becoming-activity-7349444736048320512-zjjS?utm_source=share&utm_medium=member_desktop&rcm=ACoAACdAQCkBye_l2SHKxHq1FoCJdVQJdLz5Boc)

Here’s what typically happens:

1. **PM writes**: “The assistant should be professional but approachable”
2. **Spec translates**: “Use formal language with occasional casual phrases”
3. **Engineer implements**: “You are a professional assistant. Maintain formal tone while being friendly.”
4. **Runtime adds context**: “You are a professional assistant. Maintain formal tone while being friendly. Current user: {user\_name}. Previous context: {history}”

Each step adds interpretation and delay.

**Why it matters:**

- **Speed of iteration.** Fewer hops = faster experiments.
- **Quality & intent fidelity.** Each handoff (PM → spec → UI copy → template → runtime prompt) adds interpretation risk.
- **Observability.** When prompts are hidden in code, it’s hard to debug.

**How to reduce distance:** Ask your engineer teammates to put prompts on dynamic configs, or a prompt management platform. Don’t put prompts on code.

### KV‑Cache

**What it is:** KV-Cache (Key-Value Cache) is like a smart notebook that lets LLMs remember their previous calculations. Without it, every time your chatbot generates a new word, it would need to re-read the entire conversation from scratch.

Imagine you’re having a conversation with a chatbot. Each time you send a new message, the chatbot needs to understand the entire conversation history to provide a relevant response. Without a KV-Cache, the model would have to re-read and re-process the whole conversation from the beginning every single time it generates a new word. This is incredibly inefficient and slow, leading to a frustratingly laggy user experience, especially with longer conversations or longer prompts.

![](https://cdn-images-1.medium.com/max/1024/1*h8t0jEhBl2fpC89sKmjmxw.png)

KV-Cache has two major benefits:

- **Faster Response Times (Same response qualities but faster speeds):** By avoiding redundant computations, the KV-Cache allows the LLM to generate responses much more quickly. For users, this means less waiting and a more fluid, natural interaction with your AI feature.
- **Reduced Computational Costs (Same money but more users):** Re-processing less data means using less computational power. This directly translates to lower operational costs for running your LLM, making your product more scalable.

**Why PMs should care:** modern LLM providers now charge 10x less for cached tokens than new tokens.

> For example, GPT-5 charges $1.25 for 1M input tokens, but it only charges $0.125 for 1M cached tokens.

**How to leverage KV-cache:** Always put variables at the end of your prompts.

❌ **Bad prompt structure** (minimal caching):

```
User: {{user_name}}  
Question: {{user_question}}  
Conversation history: {{chat_history}}  
  
You are a customer support agent for TechCorp.  
Guidelines:  
- Be empathetic and professional  
- Check our knowledge base before answering  
- Escalate billing issues to human agents  
- Always verify account details first
```

✅ **Good prompt structure** (maximum caching):

```
You are a customer support agent for TechCorp.  
Guidelines:  
- Be empathetic and professional    
- Check our knowledge base before answering  
- Escalate billing issues to human agents  
- Always verify account details first  
  
User: {{user_name}}  
Question: {{user_question}}  
Conversation history: {{chat_history}}
```

The static instructions get cached across all requests, while only the dynamic user content changes. For a support bot handling 10,000 daily conversations, this restructuring alone could save $200–300 or more per day.

### Structured Output / Response Schema

**What it is:** Instead of returning free‑form text, the model returns **well‑formed JSON**. You define a schema — fields, types, enums — and the model adheres to it.

**This is counterintuitive for many PMs: you don’t need to describe the format in your prompt at all.**

- **Structured output is an API-level feature**, not a prompt trick. You turn it on **in code** by registering a schema/response format; then the runtime enforces it.
- **Once it’s configured, don’t re-specify the format in the prompt.** Tell the model *what* to fill, not *how to format* it.
- **Consumer chatbot UIs (e.g., ChatGPT-style apps) generally don’t expose this.** They’re optimized for human-readable text, not machine-parsable payloads.

**We have to show some code here to explain it:**

```
import OpenAI from "openai"  
import { z } from "zod"  
import { zodTextFormat } from "openai/helpers/zod"  
  
// 1) Your schema stays in code (Zod)  
const RelevancyItem = z.object({  
  clipId: z.string(),  
  relevant: z.boolean(),  
  relevantReason: z.string(),  
  advertisement: z.boolean(),  
})  
const RelevancyArray = z.array(RelevancyItem)  
type Relevancy = z.infer<typeof RelevancyArray>  
  
// 2) Build the semantics-only prompt (no format instructions)  
const prompt = promptTemplate.join("\n").replace(INPUT_REPLACE, input)  
  
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY })  
  
// 3) Call Responses API and let the helper parse & validate for you  
const response = await openai.responses.parse({  
  model: "gpt-4o-mini",  
  input: [  
    { role: "system", content: "Judge search-trend relevancy using the definitions provided." },  
    { role: "user", content: prompt },  
  ],  
  text: {  
    // zodTextFormat drives structured output and runtime validation  
    format: zodTextFormat(RelevancyArray, "relevancy_results"),  
  },  
})  
  
// 4) Already parsed & validated:  
const results: Relevancy = response.output_parsed  
  
// 5) Same post-filtering  
const relatedResults = results.filter(  
  (r) => r.relevant && r.evergreen && !r.advertisement  
)
```

**⚠️ Conflict Notice (VERY IMPORTANT)**

Once a **schema/responseFormat** is set in code, **do not describe output formatting in the prompt**. Mixing prompt-format rules with the engineer-defined schema creates two sources of truth and measurably hurts reliability:

- **Validation failures & retries** → higher latency/cost; occasional data loss if coerced.
- **Instruction dilution** → worse task quality (model juggles format vs. content).
- **Downstream breakage** → typed logic fails on “pretty” but invalid payloads.

**Please remember:** *Schema owns format; prompt owns semantics.* Keep prompts about what each field should contain (definitions, decision rules), **not** how to format.

❌ **Your prompt should NOT say**:

```
Analyze these search results and return a JSON object with:  
- videoId: the video identifier  
- isRelevant: boolean indicating if it matches  
- relevanceReason: explanation string  
- isPaid: boolean for sponsored content  
Format as valid JSON with these exact field names.
```

✅ **Your prompt SHOULD say**:

```
Analyze these search results for relevance to the user's query.  
  
For relevance assessment:  
- Consider semantic match, not just keyword overlap  
- Educational content is preferred over entertainment  
- Recent content (last 6 months) is more relevant  
  
For paid content detection:  
- Look for "Sponsored", "Ad", or "#ad" markers  
- Check if the channel name includes "Official" or "Brand"  
  
Provide clear reasoning for why content is or isn't relevant.
```

Notice: all semantics, zero formatting. The schema handles structure; your prompt handles meaning.

One worth-to-mention common pitfall that PMs to avoid: changing field names in prompts without coordinating with engineering. If your engineer’s schema says videoId but your prompt mentions video\_id or clip\_id, you're creating confusion that degrades performance.

### Putting It All Together

![](https://cdn-images-1.medium.com/max/1024/1*qCAC6ywG17E1WbUqvhuZRg.png)

1. **Low pm to prompt distance** lets you iterate quickly on prompt improvements
2. **Optimized KV-cache** makes those iterations cheaper to test at scale
3. **Proper structured output** ensures reliable, parseable responses regardless of prompt changes

### Next Steps

1. **Audit your current setup**: Where do your prompts live? Are variables at the end? Are you mixing format and content instructions?
2. **Start one conversation**: Pick the highest-impact improvement and discuss with your engineering team. Most engineers appreciate PMs who understand these constraints.
3. **Measure the impact**: Track iteration speed, cost per request, and error rates before and after changes.

**Questions?** Drop them in the comments. Our team loves talking about efficient engineering & pm collaborations.

### Join Our Team

If these practical takeaways resonate with you and you’re passionate about solving complex technical challenges at scale, we’d love to hear from you.

Check out our open positions: [opus.pro/careers](https://www.opus.pro/careers)
