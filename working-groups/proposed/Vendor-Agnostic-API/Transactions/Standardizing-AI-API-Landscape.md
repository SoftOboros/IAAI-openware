# **Toward a Unified Interface: The IAAI "Dream API" Specification and Landscape Analysis**

## **Executive Summary**

The rapid proliferation of Large Language Models (LLMs) has precipitated a Cambrian explosion of Application Programming Interfaces (APIs), creating a fragmented technological ecosystem often described as "Patient 0" for an unkempt jungle. While the underlying capabilities of these models—text generation, reasoning, and multimodal analysis—are theoretically converging, the interfaces used to access them remain stubbornly distinct and operationally brittle. Developers integrating models from primary providers such as OpenAI, Anthropic, Google (Gemini), Cohere, and Mistral are currently forced to navigate a labyrinth of divergent schemas, authentication protocols, and error handling mechanisms. This report provides an exhaustive breakdown of the current AI API landscape, categorizing methods and common components to expose the friction points that currently inhibit true interoperability.  
The current ad-hoc approach, characterized by reactive "post-fixes," brittle wrapper libraries, and conditional logic trees that grow exponentially with each new model release, is unsustainable for enterprise-grade software development. This analysis asserts that the industry has reached a maturation point where standardization is not only beneficial but necessary for the continued scaling of AI applications. By evaluating the best and worst practices across the industry—from Cohere’s native RAG integration to Anthropic’s precise caching mechanisms and OpenAI’s structured output strictness—this report synthesizes a "Union Set" of functions into a proposed "Dream API" specification.  
This document defines specific "Levels of Compliance" for this dream specification, designed to unify the best-of-breed features into a coherent, forward-looking standard. The objective is to arm the International Association for the Advancement of Artificial Intelligence (IAAI) community with a concrete technical proposal. The ultimate goal is to transition the market dynamics from a vendor-locked "I know this beast, it’s special" mentality to a standardized utility model where the community drives vendors toward interoperable, robust, and predictable interfaces.

## ---

**1\. The Anatomy of the Jungle: A Comparative Analysis of the AI API Landscape**

The current state of AI APIs is defined by a tension between the "de facto" standard established by OpenAI's early market dominance and the divergent, proprietary approaches taken by competitors like Google, Anthropic, and Cohere. While functionality is largely overlapping, the structural implementation varies significantly, necessitating complex middleware to maintain compatibility. This section dissects these categories to reveal the underlying architectural philosophies.

### **1.1 The "De Facto" Standard: OpenAI Compatibility and Its Limits**

OpenAI’s Chat Completions API has arguably set the baseline expectation for the industry. Its structure—relying on a messages array containing objects with role and content—has been adopted by numerous open-source frameworks (e.g., vLLM, Ollama, Mistral.rs) and smaller providers (e.g., DeepSeek, Grok) to reduce friction for developers migrating from GPT models.1 The ubiquity of the openai-python library has created a gravity well where "compatibility" is often synonymous with "runs with the OpenAI SDK."  
However, strictly adhering to the OpenAI format has limitations. As models evolve to support reasoning (o1/o3 series) and complex multimodal inputs, the schema has become overloaded. For instance, image inputs use a specific image\_url structure within the content array, while audio inputs require entirely separate endpoints or the new Realtime API WebSocket protocol, fracturing the developer experience.4 Furthermore, parameters like logit\_bias and frequency\_penalty are not consistently supported across all "OpenAI-compatible" endpoints, leading to runtime errors when developers assume full parity.6 The "compatibility" is often skin-deep; while the JSON shape matches, the semantic behavior of parameters like temperature or system prompts can vary wildly between models using the same interface.

### **1.2 The Divergent Giant: Google Gemini (Vertex AI)**

Google’s Gemini API represents the most significant deviation from the norm. Stemming from a lineage of Protocol Buffers and gRPC-centric design within Google’s internal infrastructure, the REST representation of Gemini’s API differs fundamentally from the OpenAI schema. Instead of a messages array, Gemini uses a contents array; instead of role: assistant, it uses role: model; and rather than a simple text string, it employs a parts array that strictly separates text from inline data blobs.8  
This structure, while logically sound for a multimodal-native model where text and images are treated as equal data "parts," creates immense friction for interoperability. A developer cannot simply swap an endpoint URL; they must fundamentally restructure the payload. For example, Gemini’s handling of system instructions is often separated into a distinct system\_instruction field rather than being a message with role: system, a distinction that requires specific branching logic in any integration layer.10 Furthermore, the error handling in Google’s ecosystem is notably verbose, often returning 400 errors for schema violations that other APIs might handle more gracefully or describe differently, often citing specific protocol buffer violations.11

### **1.3 The Safety-First Architect: Anthropic**

Anthropic’s API design reflects its prioritization of safety, interpretability, and strict adherence to prompt structures. Historically, it enforced strict alternating user/assistant turns, a constraint that other providers are looser with. The Anthropic messages API separates the system prompt into a top-level parameter, distinct from the conversation history messages array. This architectural choice avoids the ambiguity of injecting system instructions as "fake" user messages, which is a common hack in OpenAI-based implementations.13  
Anthropic also pioneered the concept of explicit content blocks early on, allowing for mixed text and tool use in a single turn. Their recent introduction of "Thinking" blocks (for models that reason before answering) and detailed usage of XML tags for guidance set them apart.13 While their move toward a more JSON-centric "tool use" structure brings them closer to the industry norm, the underlying philosophy of explicit, structured separation remains a distinct architectural choice. This design forces developers to be more intentional about state management, particularly regarding the cache\_control headers used for their prompt caching features.13

### **1.4 The Enterprise Specialist: Cohere**

Cohere differentiates itself by embedding Retrieval Augmented Generation (RAG) directly into the API contract. Unlike others where RAG is an external orchestration step managed by the client, Cohere’s chat endpoint includes a documents parameter and connectors for web search.16 This allows the model to perform citations and grounding natively, returning response objects that interweave generated text with fine-grained citation metadata.  
This "RAG-first" design is structurally superior for enterprise search applications, as it offloads the complexity of citation mapping to the model provider. However, it represents a "special beast" that requires specific handling in a unified interface. While other APIs return simple text or tool calls, Cohere returns a complex object containing citations, documents, and search\_results, creating a unique response schema that does not map 1:1 with GPT or Claude responses.17

### **1.5 The Open Frontier: Mistral and Meta LLaMA**

Mistral and Meta (via hosting providers like Groq, Together, or Lepton) largely adhere to the OpenAI format but introduce subtle variations that can break integrations. Mistral, for instance, supports a safe\_prompt boolean to enforce guardrails at the API level, a feature not present in the standard OpenAI spec.19  
Meta’s LLaMA models, when accessed via raw endpoints or self-hosted environments (e.g., using vLLM), often leak the underlying prompt templating requirements (e.g., \<|begin\_of\_text|\>, \`\`) through to the API layer. While standard wrappers attempt to abstract this, the "leakage" of tokenization nuances forces developers to manage model-specific string formatting manually to achieve optimal performance, negating the benefit of a standardized JSON interface.20

### **Table 1: Comparative Landscape of AI API Architectures**

| Feature Domain | OpenAI (GPT-4o/o1) | Google (Gemini 1.5/2.0) | Anthropic (Claude 3.5) | Cohere (Command R+) | Mistral (Large) |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Auth Protocol** | Bearer Token | OAuth / API Key (Header) | x-api-key / AWS SigV4 | Bearer Token | Bearer Token |
| **Message Container** | messages (role/content) | contents (role/parts) | messages \+ system param | chat\_history \+ message | messages |
| **Role Names** | system, user, assistant | user, model | user, assistant | USER, CHATBOT | system, user, assistant |
| **RAG/Grounding** | Client-side / Assistants API | Client-side / Vertex Search | Client-side | Native documents param | Client-side |
| **Reasoning Visibility** | Hidden (o1 tokens) | thinking content block | Hidden | N/A | Hidden |
| **JSON Output** | response\_format (schema) | responseMimeType | tool\_use / output\_format | response\_format | response\_format |
| **Multimodal Input** | image\_url object | inline\_data (blob) / file\_data | source (base64) | No (Text focus) | image\_url |

## ---

**2\. Core Interaction Primitives: The Need for Standardization**

To construct the "Dream API," one must dissect the existing methods to identify the atomic units of interaction. The friction in the current landscape arises not just from different field names, but from fundamentally different conceptual models of how an AI interaction should flow.

### **2.1 Authentication and Connection Protocols: The Gatekeeper Problem**

The "jungle" begins at the very front door: authentication.

* **Bearer Tokens:** OpenAI, Anthropic, Mistral, and Cohere use standard Authorization: Bearer \<token\> headers. This is the path of least resistance and the industry preference for ease of use.22  
* **Legacy and Enterprise Complexity:** Google allows simple API keys for its AI Studio (prototyping) environment but demands complex OAuth 2.0 flows via gcloud authentication for production Vertex AI workloads. This effectively bifurcates the implementation code, forcing developers to write two distinct authentication handlers for the same model family depending on the deployment environment.24  
* **Cryptographic Overhead:** Accessing models via aggregators like Amazon Bedrock requires AWS Signature Version 4 (SigV4) signing. This heavy cryptographic process prevents simple HTTP requests (like curl) without robust SDKs, significantly raising the barrier to entry for lightweight integrations or edge devices.26

**Insight:** The "Dream API" must mandate a standardized Bearer token approach for simplicity in standard interactions, while supporting OIDC-compliant identity federation for enterprise security. This decouples the transport authentication from the provider's internal infrastructure complexity, allowing the client to remain agnostic to whether the model is hosted on Google Cloud, AWS, or a private server.

### **2.2 The Request Object: The Message Container**

The core atomic unit of an LLM interaction is the Message. However, the schema for this object is inconsistent.

* **The Overloaded Content String:** OpenAI and Mistral use \[{"role": "user", "content": "..."}\]. This is simple but becomes overloaded when multimodal data is introduced. Complex objects containing image\_url or text are stuffed into the content field, requiring conditional parsing logic on the receiver side.27  
* **The Block-Based Approach:** Anthropic uses a messages list plus a separate system parameter. The content field is strictly a list of blocks (text, image, tool\_use). This explicitness prevents ambiguity and makes parsing stream events significantly more reliable.13  
* **The Blob Approach:** Gemini uses contents arrays of objects with parts. These parts can contain text or inline\_data (blobs). This blob structure is efficient for raw bytes but harder to debug than base64 strings, and the field naming (parts vs content) breaks every standard JSON serialization library used for other LLMs.8

**The Union Requirement:** A robust API must support a messages array where content is strictly typed as an array of blocks. It must allow for text, image, audio, video, and file parts, accommodating both inline base64 (for speed/simplicity) and external URLs/URIs (for efficiency and caching). The system prompt should be elevated to a top-level parameter to resolve the ambiguity of its role in the conversation history.4

### **2.3 Structured Outputs and Tool Use: From Hints to Contracts**

This area is currently the most chaotic sector of the API landscape.

* **Evolution of JSON:** OpenAI originally used function\_call, then deprecated it for tools, and now offers response\_format: {type: "json\_object"}. Most recently, they introduced Strict Structured Outputs using rigid JSON Schema validation.30  
* **Type Safety:** Anthropic initially relied on XML parsing but has moved to tool\_choice and input\_schema. Their "Strict" mode enforces exact schema adherence, ensuring that the model does not hallucinate parameters.15  
* **Configuration sprawl:** Gemini uses functionDeclarations inside a tools object. While it supports responseMimeType: "application/json" and schema validation, the configuration is often buried deep within a generationConfig object, separating the schema definition from the logical intent of the request.11  
* **Validation Gaps:** Mistral supports response\_format similar to OpenAI but has historically offered weaker validation guarantees, sometimes returning markdown-wrapped JSON that requires post-processing cleanup.33

**Critical Insight:** The "Dream API" must elevate Structured Output from a probabilistic "hint" to a deterministic contract. It should support standard JSON Schema (draft 2020-12) natively, including recursive types, and provide explicit validation error feedback when the model fails to generate compliant JSON, rather than returning a hallucinated string that breaks downstream parsers.32

### **2.4 Streaming Architectures: The Latency Lifeline**

Streaming is essential for perceived latency in AI applications, but implementation variations create significant technical debt.

* **Server-Sent Events (SSE):** This is the industry standard transport, but the payload structure varies.  
* **Deltas vs. Events:** OpenAI streams deltas—tiny fragments of text that must be concatenated by the client. While efficient for text, this creates state management nightmares for tool calling, as a single tool call might be split across twenty chunks.35  
* **State-Aware Blocks:** Anthropic streams explicit events (content\_block\_start, content\_block\_delta, content\_block\_stop). This allows for cleaner handling of mixed content types (e.g., streaming text and tool calls simultaneously) and provides a state machine that the client can implement reliably.36  
* **Aggregation Ambiguity:** Gemini streams GenerateContentResponse chunks which may contain partial text or aggregation of citations. The lack of a clear "delta" field sometimes forces clients to reconstruct the whole message state from cumulative chunks, increasing client-side computational overhead.9

**Insight:** Anthropic’s event-based streaming model is the most robust for complex agentic workflows, as it explicitly delineates between reasoning, tool usage, and final text generation. OpenAI’s delta model is simpler for pure text but brittle for complex multi-step generations. The Dream API must adopt an event-based stream that clearly signals the start and end of semantic blocks.

## ---

**3\. The Control Plane: Tools, Logic, and Reasoning**

Beyond the basic request/response cycle, the "Control Plane" of AI APIs governs how models think, use tools, and adhere to strict logical constraints. This is where the divergence between providers transitions from syntactic to semantic.

### **3.1 The Tooling Divergence: Function Calling vs. Execution**

The concept of "tool use" (formerly function calling) allows LLMs to interact with the outside world. However, the implementation models differ fundamentally.

* **The Orchestrator Model (OpenAI/Mistral):** The model returns a tool call request (JSON). The client executes the code and sends the result back in a new message with role: tool. The model then generates the final response. This puts the burden of execution and safety entirely on the client.27  
* **The Execution Model (Google/Anthropic):** Google’s Gemini and Anthropic (via Code Interpreter) represent a shift toward server-side execution. Gemini can execute Python code in a sandboxed environment on Google’s servers and return the *result* directly to the model context, skipping the client-side round trip entirely for logic tasks. Anthropic’s specialized tools allow for similar "computer use" paradigms.8

**Insight:** The Dream API must support both paradigms. It requires a tools definition that allows for client-side execution (standard) but also a capabilities flag to enable server-side sandboxes (e.g., capability: code\_interpreter), abstracting the execution environment from the prompt engineering.

### **3.2 Reasoning and "Thinking" Blocks**

With the advent of reasoning models like OpenAI’s o1 and DeepSeek R1, the internal "Chain of Thought" has become a critical component of the interaction.

* **Hidden Reasoning:** OpenAI currently hides the reasoning tokens for the o1 model, returning only the final answer. This creates a "black box" that limits debuggability and trust for enterprise users who need to verify the logic path.1  
* **Visible Reasoning:** DeepSeek and newer open models expose the reasoning trace as unstructured text, often polluting the final answer or requiring regex parsing to separate "thoughts" from "response."  
* **Structured Reasoning:** Anthropic has introduced explicit thinking blocks in their newer beta specs. This is the architecturally correct choice: it separates the *process* from the *result* structurally.

**Dream API Requirement:** The Dream API must mandate a thinking content block type. This allows the API consumer to choose whether to display, log, or discard the reasoning trace, providing transparency without compromising the structure of the final output.36

### **3.3 The JSON Schema Wars**

Validating output is critical for agentic workflows. The industry is currently split between different schema standards.

* **OpenAPI vs. Pydantic:** OpenAI heavily leverages Pydantic for Python-based schema generation, while Google enforces a subset of the OpenAPI 3.0 schema standard.11  
* **Strictness:** OpenAI’s "Strict Mode" and Anthropic’s "Strict Tool Use" both solve the problem of hallucinated parameters, but they do so via different API flags (strict: true vs tool\_choice: "any").  
* **The Union:** The Dream API should adopt the standard JSON Schema Draft 2020-12 as the canonical definition language, supporting advanced features like recursive definitions which are currently patchy across providers.37

## ---

**4\. The Sensorium: Multimodal Inputs and Outputs**

The era of text-only LLMs is over. The "Unkempt Jungle" is perhaps densest in the handling of non-text modalities.

### **4.1 Vision: Blobs vs. URLs**

Handling images requires balancing security, performance, and convenience.

* **The URL Convenience:** OpenAI’s image\_url allows passing a publicly accessible URL. This is developer-friendly but introduces latency (the model must fetch the image) and privacy concerns.4  
* **The Base64 Reality:** For private images, base64 encoding is the standard fallback. However, this bloats the JSON payload significantly, risking timeouts or payload size limits on standard HTTP proxies.  
* **The Cloud Storage Solution:** Google Gemini allows file\_data referencing a Google Cloud Storage URI. This is efficient for massive files (video/high-res images) but locks the user into the Google ecosystem.38

**Recommendation:** The Dream API must support a unified content\_block that accepts source objects. This source object must support type: base64, type: url, and crucially, type: s3\_presigned or type: storage\_ref to allow for secure, cloud-agnostic file referencing without payload bloat.

### **4.2 Audio: The Protocol Schism**

Audio represents the deepest fracture in the landscape.

* **RESTful Audio:** Google Gemini treats audio as just another "part" of the message. You can upload an MP3 and ask questions about it in the same generateContent call. This is "native" multimodality.40  
* **Segregated Endpoints:** OpenAI historically separated audio into audio/transcriptions (Whisper) and audio/speech (TTS). To build a voice bot, a developer had to chain these calls manually, incurring massive latency.42  
* **The WebSocket Frontier:** To solve latency, OpenAI introduced the Realtime API over WebSockets. While performant, this abandons the REST paradigm entirely, requiring a completely different set of tools and error handling logic.5

**Dream API Requirement:** The specification must define a modality negotiation. For non-realtime interactions, audio should be a standard content block (like Gemini). For realtime, the API must define a standard WebSocket sub-protocol (based on the Dream API schema) to prevent the "Realtime" ecosystem from diverging into proprietary binary formats.

## ---

**5\. The Operational Layer: Context, Tuning, and Errors**

A standardized API is not just about the happy path; it is about how the system behaves when things go wrong or when it needs to be customized.

### **5.1 RAG and Context Management**

Current APIs (OpenAI) require developers to "stuff" RAG context into the system prompt or user message. This is semantically incorrect; context is not an "instruction" (system) nor a "query" (user). It is reference material.

* **The Cohere Model:** Cohere’s approach of a dedicated documents field allows the engine to treat this data specially—potentially applying distinct attention masks or citation logic.16  
* **Caching:** Anthropic introduced explicit cache\_control headers on message blocks. This allows developers to mark static context (like a 50-page manual) to be cached at the edge, drastically reducing costs and latency.13

**Union Proposal:** The Dream API must include a top-level context object in the request, distinct from messages. This object handles documents (RAG) and supports cache\_control flags, standardizing the optimization of input tokens.

### **5.2 Fine-Tuning and Model Adaptation**

Fine-tuning remains a fragmented operational workflow.

* **Data Formats:** OpenAI mandates JSONL (JSON Lines) formatted specifically for chat completions.43 Vertex AI (Gemini) often requires data to be staged in buckets with specific directory structures, adding operational overhead.44  
* **Hyperparameters:** The naming conventions for learning rate multipliers, epochs, and batch sizes vary, making it difficult to port a training job from one provider to another without re-tuning the configuration.46

### **5.3 Standardization of Error Codes**

The current landscape is a minefield of HTTP 400s.

* *Current:* OpenAI returns context\_length\_exceeded strings. Google returns generic 400 Bad Request responses with nested protocol buffer error messages that are difficult to parse programmatically.12  
* *Dream Standard:* The industry must adopt **RFC 7807 (Problem Details for HTTP APIs)**.  
  * type: "[https://api.dream.org/errors/context-limit](https://api.dream.org/errors/context-limit)"  
  * title: "Context Window Exceeded"  
  * detail: "Request required 5000 tokens, limit is 4096."  
  * extension: { "limit": 4096, "requested": 5000 } (Machine readable\!).48

## ---

**6\. The "Dream API" Specification: Levels of Compliance**

Based on the exhaustive analysis above, we propose the "Dream API" Specification. This is not merely a wrapper but a formal definition of "Levels of Compliance" that vendors should aspire to. It uses the best features of current art to create a superset of functionality.

### **6.1 Core Philosophy**

The Dream API is **Resource-Oriented**, **State-Aware**, and **Natively Multimodal**. It abandons the text-centric "chat" paradigm in favor of an "Interaction" paradigm that encompasses all modalities and tools.

### **6.2 Compliance Levels**

#### **Level 1: Foundation (The Universal Text Interface)**

*Requirement:* A provider must support a unified messages endpoint for text-in/text-out.

* **Endpoint:** POST /v1/interactions  
* **Schema:**  
  * messages: Array of {role, content}.  
  * role: Strictly system, user, assistant.  
  * content: String or array of text blocks.  
  * model: String ID.  
* **Output:** Standardized JSON with id, created, and content.  
* **Error Handling:** Must return RFC 7807 compliant error objects.

#### **Level 2: Structured & Tooling (The Agentic Interface)**

*Requirement:* Support for strict JSON output and deterministic tool calling.

* **Features:**  
  * **Tool Definitions:** A tools array using standard JSON Schema (Draft 2020-12).  
  * **Tool Choice:** Explicit control (auto, required, none, or specific tool).  
  * **Structured Output:** A response\_schema parameter that *guarantees* adherence to a schema (Pydantic-style validation built-in).32  
  * **Reasoning:** Support for explicit thinking content blocks.

#### **Level 3: Multimodal & Contextual (The Omni Interface)**

*Requirement:* Native handling of non-text inputs/outputs and external context.

* **Features:**  
  * **Unified Content Blocks:** content array accepts text, image, audio, video, and file.  
  * **Context/RAG:** A documents parameter (Cohere style) allowing the user to pass raw text chunks or citations that the model *must* use for grounding, returning citations in the response.16  
  * **Caching:** Explicit cache\_control flags on message blocks.

#### **Level 4: Realtime & Stateful (The Living Interface)**

*Requirement:* WebSocket/SSE interfaces for low-latency, interruptible streams.

* **Features:**  
  * **Bi-directional Streaming:** Full duplex communication for voice-to-voice.  
  * **Session Management:** session\_id to maintain state server-side, reducing the need to re-transmit the entire context window for every turn (efficiency gain).51

### **6.3 Proposed Schema: The "Union" Request Object**

The following JSON specification represents the "Dream API" request body, combining the best attributes of OpenAI, Anthropic, and Cohere.

JSON

{  
  "model": "vendor-model-id",  
  "session\_id": "optional-server-side-state-id",  
  "config": {  
    "temperature": 0.7,  
    "max\_tokens": 4096,  
    "safety\_mode": "strict", // Mistral style  
    "stream": true  
  },  
  "context": {  
    "system": "You are a helpful assistant...", // Anthropic style separation  
    "documents":,  
    "citations\_required": true  
  },  
  "messages": \[  
    {  
      "role": "user",  
      "content": \[  
        { "type": "text", "text": "Analyze this chart." },  
        {   
          "type": "image",   
          "source": { "type": "url", "url": "https://..." }, // OpenAI/Anthropic union  
          "cache\_control": { "type": "ephemeral" } // Anthropic caching  
        }  
      \]  
    }  
  \],  
  "tools": {  
    "definitions": \[...\], // Standard JSON Schema  
    "choice": "auto",  
    "strict\_validation": true // OpenAI/Anthropic strict mode  
  },  
  "response\_format": {  
    "type": "json\_schema", // Gemini/OpenAI union  
    "schema": {...}   
  }  
}

### **6.4 Proposed Schema: The "Union" Response Object**

The response normalizes the chaotic "delta" vs "part" situation.

JSON

{  
  "id": "msg\_123",  
  "created": 1700000000,  
  "usage": { "input": 50, "output": 100, "cache\_read": 200 }, // Detailed metrics  
  "choices": \[  
    {  
      "finish\_reason": "stop", // or tool\_calls, content\_filter  
      "citations": \[ // Cohere style  
        { "start\_index": 5, "end\_index": 20, "document\_ids": \["doc1"\] }  
      \],  
      "message": {  
        "role": "assistant",  
        "content":  
      }  
    }  
  \]  
}

## ---

**7\. Implementation Strategy: From Jungle to Garden**

The transition from the current jungle to this standardized garden cannot be forced; it must be engineered through community consensus and tooling.

### **7.1 The "Shim" Strategy**

The IAAI should sponsor an open-source reference implementation (a "Dream Shim") that sits on top of LiteLLM or similar libraries. This shim would expose the Dream API specification and internally translate it to OpenAI, Anthropic, and Gemini formats. This proves viability and allows developers to code against the standard immediately.3

### **7.2 The "Compliance Badge"**

We propose a certification program: **"IAAI Level 2 Compliant"**. Vendors can market their compliance to attract enterprise customers who value portability. This shifts the narrative from "OpenAI-compatible" (which cements OpenAI's idiosyncrasies) to "Standard-Compliant" (which creates a neutral playing field).

### **7.3 Deprecation of "Chat" for "Interaction"**

We must stop calling these "Chat APIs." "Chat" implies text-only, turn-based human conversation. These are **Inference Engines**. The Dream API nomenclature uses interaction or completion to welcome non-chat use cases like batch processing, image generation, and autonomous agent loops without semantic dissonance.

## ---

**Conclusion**

The "jungle" of AI APIs is not a result of necessary innovation, but of uncoordinated evolution. We have analyzed the landscape—from OpenAI’s robust but aging schema to Gemini’s complex multimodal structures and Anthropic’s rigid safety protocols—and found that no single provider offers the perfect interface. However, the pieces of the perfect interface exist, scattered across them all.  
The "Dream API" proposed here is not rocket science; it is a rational consolidation of best practices. By standardizing around a union set of features—native RAG, structured reasoning, multimodal content blocks, and rigorous error handling—we can elevate the entire industry. The IAAI has the opportunity to lead this discussion, moving the community from maintaining fragile "post-fixes" to building on a solid, unified foundation. The beast is not special; it is just a database of probabilities. It is time we talked to it in a common language.

### **Table 2: Feature Matrix \- Current Landscape vs. Dream API**

| Feature | OpenAI | Anthropic | Gemini (Vertex) | Cohere | Dream API Proposal |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Auth** | Bearer Token | x-api-key / AWS SigV4 | OAuth / x-goog-api-key | Bearer Token | **Bearer \+ OIDC** |
| **Message Structure** | messages (role/content) | messages \+ top-level system | contents (role/parts) | chat\_history | **Unified messages with typed blocks** |
| **RAG/Grounding** | No native param (Tools only) | No native param | tools (grounding) | documents \+ connectors | **Native documents param & citations** |
| **JSON Output** | response\_format (schema) | tool\_use (schema) | responseMimeType | response\_format | **Native response\_schema (Draft 2020\)** |
| **Multimodal** | image\_url object | source (base64) | inline\_data / file\_data | No | **Unified content\_block (url/base64/file)** |
| **Streaming** | Deltas (text fragments) | Events (block state) | Response chunks | Stream events | **Event-based (Block State)** |
| **Reasoning** | Hidden (o1) | thinking block | Hidden | N/A | **Explicit thinking blocks** |

#### **Works cited**

1. LLM API Pricing Comparison (2025): OpenAI, Gemini, Claude | IntuitionLabs, accessed December 21, 2025, [https://intuitionlabs.ai/articles/llm-api-pricing-comparison-2025](https://intuitionlabs.ai/articles/llm-api-pricing-comparison-2025)  
2. Structured Output Comparison across popular LLM providers — OpenAI, Gemini, Anthropic, Mistral and AWS Bedrock | by Rost Glukhov | Medium, accessed December 21, 2025, [https://medium.com/@rosgluk/structured-output-comparison-across-popular-llm-providers-openai-gemini-anthropic-mistral-and-1a5d42fa612a](https://medium.com/@rosgluk/structured-output-comparison-across-popular-llm-providers-openai-gemini-anthropic-mistral-and-1a5d42fa612a)  
3. LiteLLM \- AI/ML API Documentation, accessed December 21, 2025, [https://docs.aimlapi.com/integrations/litellm](https://docs.aimlapi.com/integrations/litellm)  
4. Images and vision | OpenAI API, accessed December 21, 2025, [https://platform.openai.com/docs/guides/images-vision](https://platform.openai.com/docs/guides/images-vision)  
5. Realtime conversations | OpenAI API, accessed December 21, 2025, [https://platform.openai.com/docs/guides/realtime-conversations](https://platform.openai.com/docs/guides/realtime-conversations)  
6. Completions | OpenAI API Reference, accessed December 21, 2025, [https://platform.openai.com/docs/api-reference/completions](https://platform.openai.com/docs/api-reference/completions)  
7. HTTP server \- mistral.rs \- GitHub, accessed December 21, 2025, [https://github.com/EricLBuehler/mistral.rs/blob/master/docs/HTTP.md](https://github.com/EricLBuehler/mistral.rs/blob/master/docs/HTTP.md)  
8. Generating content | Gemini API \- Google AI for Developers, accessed December 21, 2025, [https://ai.google.dev/api/generate-content](https://ai.google.dev/api/generate-content)  
9. Gemini API reference | Google AI for Developers, accessed December 21, 2025, [https://ai.google.dev/api](https://ai.google.dev/api)  
10. Text generation | Gemini API \- Google AI for Developers, accessed December 21, 2025, [https://ai.google.dev/gemini-api/docs/text-generation](https://ai.google.dev/gemini-api/docs/text-generation)  
11. Structured output | Generative AI on Vertex AI \- Google Cloud Documentation, accessed December 21, 2025, [https://docs.cloud.google.com/vertex-ai/generative-ai/docs/multimodal/control-generated-output](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/multimodal/control-generated-output)  
12. Troubleshooting guide | Gemini API \- Google AI for Developers, accessed December 21, 2025, [https://ai.google.dev/gemini-api/docs/troubleshooting](https://ai.google.dev/gemini-api/docs/troubleshooting)  
13. Anthropic Chat Messages \- New API, accessed December 21, 2025, [https://doc.newapi.pro/en/api/anthropic-chat/](https://doc.newapi.pro/en/api/anthropic-chat/)  
14. Adapting Your Messages Array for Any LLM API: A Developer's Guide \- Jeffrey Taylor, accessed December 21, 2025, [https://jeftaylo.medium.com/adapting-your-messages-array-for-any-llm-api-a-developers-guide-5fe5b54086fa](https://jeftaylo.medium.com/adapting-your-messages-array-for-any-llm-api-a-developers-guide-5fe5b54086fa)  
15. Introducing advanced tool use on the Claude Developer Platform \- Anthropic, accessed December 21, 2025, [https://www.anthropic.com/engineering/advanced-tool-use](https://www.anthropic.com/engineering/advanced-tool-use)  
16. Chat \- Cohere Documentation, accessed December 21, 2025, [https://docs.cohere.com/reference/chat](https://docs.cohere.com/reference/chat)  
17. Building Production AI Apps with Cohere API: A Developer's Guide \- DhiWise, accessed December 21, 2025, [https://www.dhiwise.com/post/building-production-ai-apps-cohere-api](https://www.dhiwise.com/post/building-production-ai-apps-cohere-api)  
18. Using the Cohere Chat API for Text Generation, accessed December 21, 2025, [https://docs.cohere.com/docs/chat-api](https://docs.cohere.com/docs/chat-api)  
19. Usage | Mistral Docs, accessed December 21, 2025, [https://docs.mistral.ai/capabilities/completion/usage](https://docs.mistral.ai/capabilities/completion/usage)  
20. AI APIs Compared: Which Provider Delivers Best Value in 2025? | The AI Journal, accessed December 21, 2025, [https://aijourn.com/ai-apis-compared-which-provider-delivers-best-value-in-2025/](https://aijourn.com/ai-apis-compared-which-provider-delivers-best-value-in-2025/)  
21. OpenAI API chat/completions endpoint \- OpenVINO™ documentation, accessed December 21, 2025, [https://docs.openvino.ai/2024/openvino-workflow/model-server/ovms\_docs\_rest\_api\_chat.html](https://docs.openvino.ai/2024/openvino-workflow/model-server/ovms_docs_rest_api_chat.html)  
22. REST API Reference | xAI, accessed December 21, 2025, [https://docs.x.ai/docs/api-reference](https://docs.x.ai/docs/api-reference)  
23. API Specs \- Mistral AI Documentation, accessed December 21, 2025, [https://docs.mistral.ai/api](https://docs.mistral.ai/api)  
24. Gemini API in Vertex AI quickstart \- Google Cloud Documentation, accessed December 21, 2025, [https://docs.cloud.google.com/vertex-ai/generative-ai/docs/start/quickstart](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/start/quickstart)  
25. Google Gen AI SDK | Generative AI on Vertex AI \- Google Cloud Documentation, accessed December 21, 2025, [https://docs.cloud.google.com/vertex-ai/generative-ai/docs/sdks/overview](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/sdks/overview)  
26. Providers \- LiteLLM, accessed December 21, 2025, [https://docs.litellm.ai/docs/providers](https://docs.litellm.ai/docs/providers)  
27. Chat Completions | OpenAI API Reference \- OpenAI Platform, accessed December 21, 2025, [https://platform.openai.com/docs/api-reference/chat](https://platform.openai.com/docs/api-reference/chat)  
28. Anthropic.Messages.Request — anthropic\_community v0.4.3 \- Hexdocs, accessed December 21, 2025, [https://hexdocs.pm/anthropic\_community/Anthropic.Messages.Request.html](https://hexdocs.pm/anthropic_community/Anthropic.Messages.Request.html)  
29. Using OpenAI libraries with Vertex AI \- Google Cloud Documentation, accessed December 21, 2025, [https://docs.cloud.google.com/vertex-ai/generative-ai/docs/migrate/openai/overview](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/migrate/openai/overview)  
30. The guide to structured outputs and function calling with LLMs \- Agenta, accessed December 21, 2025, [https://agenta.ai/blog/the-guide-to-structured-outputs-and-function-calling-with-llms](https://agenta.ai/blog/the-guide-to-structured-outputs-and-function-calling-with-llms)  
31. Structured outputs \- Claude Docs, accessed December 21, 2025, [https://platform.claude.com/docs/en/build-with-claude/structured-outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)  
32. Structured Outputs | Gemini API \- Google AI for Developers, accessed December 21, 2025, [https://ai.google.dev/gemini-api/docs/structured-output](https://ai.google.dev/gemini-api/docs/structured-output)  
33. Enforcing JSON Outputs in Commercial LLMs \- DataChain, accessed December 21, 2025, [https://datachain.ai/blog/enforcing-json-outputs-in-commercial-llms](https://datachain.ai/blog/enforcing-json-outputs-in-commercial-llms)  
34. Ocr Endpoints \- Mistral AI Documentation, accessed December 21, 2025, [https://docs.mistral.ai/api/endpoint/ocr](https://docs.mistral.ai/api/endpoint/ocr)  
35. API Streaming | Real-time Model Responses in OpenRouter, accessed December 21, 2025, [https://openrouter.ai/docs/api/reference/streaming](https://openrouter.ai/docs/api/reference/streaming)  
36. Streaming Messages \- Claude Docs, accessed December 21, 2025, [https://docs.anthropic.com/en/api/messages-streaming](https://docs.anthropic.com/en/api/messages-streaming)  
37. Improving Structured Outputs in the Gemini API \- Google Blog, accessed December 21, 2025, [https://blog.google/technology/developers/gemini-api-structured-outputs/](https://blog.google/technology/developers/gemini-api-structured-outputs/)  
38. Generate content with the Gemini API in Vertex AI \- Google Cloud Documentation, accessed December 21, 2025, [https://docs.cloud.google.com/vertex-ai/generative-ai/docs/model-reference/inference](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/model-reference/inference)  
39. Get started with Gemini's multimodal capabilities \- Patrick Loeber, accessed December 21, 2025, [https://patloeber.com/gemini-multimodal/](https://patloeber.com/gemini-multimodal/)  
40. Analyze audio files using the Gemini API | Firebase AI Logic \- Google, accessed December 21, 2025, [https://firebase.google.com/docs/ai-logic/analyze-audio](https://firebase.google.com/docs/ai-logic/analyze-audio)  
41. Audio understanding | Gemini API \- Google AI for Developers, accessed December 21, 2025, [https://ai.google.dev/gemini-api/docs/audio](https://ai.google.dev/gemini-api/docs/audio)  
42. Audio and speech | OpenAI API, accessed December 21, 2025, [https://platform.openai.com/docs/guides/audio](https://platform.openai.com/docs/guides/audio)  
43. Customize a model with Microsoft Foundry fine-tuning \- Azure OpenAI, accessed December 21, 2025, [https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/fine-tuning?view=foundry-classic](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/how-to/fine-tuning?view=foundry-classic)  
44. Tuning | Gemini API \- Google AI for Developers, accessed December 21, 2025, [https://ai.google.dev/api/tuning](https://ai.google.dev/api/tuning)  
45. Permissions | Gemini API | Google AI for Developers, accessed December 21, 2025, [https://ai.google.dev/api/tuning/permissions](https://ai.google.dev/api/tuning/permissions)  
46. Fine-tuning best practices | OpenAI API, accessed December 21, 2025, [https://platform.openai.com/docs/guides/fine-tuning-best-practices](https://platform.openai.com/docs/guides/fine-tuning-best-practices)  
47. HTTP status and error codes for JSON | Cloud Storage, accessed December 21, 2025, [https://docs.cloud.google.com/storage/docs/json\_api/v1/status-codes](https://docs.cloud.google.com/storage/docs/json_api/v1/status-codes)  
48. Best Practices for Consistent API Error Handling | Zuplo Learning Center, accessed December 21, 2025, [https://zuplo.com/learning-center/best-practices-for-api-error-handling](https://zuplo.com/learning-center/best-practices-for-api-error-handling)  
49. Errors Best Practices in REST API Design \- Speakeasy, accessed December 21, 2025, [https://www.speakeasy.com/api-design/errors](https://www.speakeasy.com/api-design/errors)  
50. A Hands-On Guide to Anthropic's New Structured Output Capabilities, accessed December 21, 2025, [https://towardsdatascience.com/hands-on-with-anthropics-new-structured-output-capabilities/](https://towardsdatascience.com/hands-on-with-anthropics-new-structured-output-capabilities/)  
51. Designing APIs for LLM Apps: Build Scalable and AI-Ready Interfaces \- Gravitee, accessed December 21, 2025, [https://www.gravitee.io/blog/designing-apis-for-llm-apps](https://www.gravitee.io/blog/designing-apis-for-llm-apps)