# Note 16: Gemini Integration & API Schemas

## 1. Concept Overview
**Google Gemini Developer API** is a high-performance, developer-focused API interface for accessing Google's Gemini family of models (such as `gemini-1.5-flash` and `gemini-1.5-pro`).

By integrating directly with Gemini, we query the model using its native content structure (roles, parts, systemInstruction) rather than translating requests through third-party gateways. This reduces latency, increases stability, and enables the use of Gemini's massive context windows and advanced multimodal capabilities.

---

## 2. Why It Exists
Integrating directly with Gemini API offers:
1. **Low Latency**: Directly calling Google's endpoints minimizes networking hops.
2. **Advanced Context Support**: Gemini handles native structures like system instructions separately from chat messages, ensuring guidelines are followed strictly.
3. **Structured Schemas**: A standardized JSON format for representing user and model turns, keeping communications highly structured.

---

## 3. Real-World Analogy
Imagine a direct phone line to the President of a company:
- **With a gateway**: You call a central switchboard (gateway). They translate your query, routing it through multiple internal operators before reaching the President. If the switchboard goes down, you lose the call.
- **With Gemini Direct**: You dial the President's personal desk phone directly. It rings instantly, and they answer without middle-layer translations.

---

## 4. Technical Explanation
The Gemini API organizes chat messages into a list of `contents` objects, where roles must be either `"user"` or `"model"`. System prompts are defined outside the message history under `systemInstruction`:

```json
{
  "contents": [
    {
      "role": "user",
      "parts": [{"text": "Who signed the contract?"}]
    }
  ],
  "systemInstruction": {
    "parts": [{"text": "You are a grounding assistant. Answer ONLY using the facts..."}]
  },
  "generationConfig": {
    "temperature": 0.0
  }
}
```

### Direct Python HTTP Client:
We communicate with the endpoints using `httpx`:
- **Sync URL**: `https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent?key={API_KEY}`
- **SSE Stream URL**: `https://generativelanguage.googleapis.com/v1beta/models/{model}:streamGenerateContent?alt=sse&key={API_KEY}`

Streaming returns Server-Sent Events (SSE) where each data chunk is a candidate JSON block containing generated parts:

```python
# Streaming chunk parsing
chunk_data = json.loads(line)
token = chunk_data["candidates"][0]["content"]["parts"][0].get("text", "")
```

---

## 5. Industry Best Practices
- **Separate System Instructions**: Always pass your grounding prompt inside the `systemInstruction` block rather than appending it as a chat bubble. Gemini models are trained to prioritize instructions sent in this node.
- **Translate Roles Carefully**: Ensure your database conversation logs (often using standard OpenAI schemas like "assistant") translate "assistant" to "model" before invoking Gemini.
- **Pin the API Version**: Use stable endpoints (e.g., `v1beta` or `v1`) in your URLs to prevent breaking changes when Google updates their API structures.

---

## 6. Common Mistakes
- **Passing Unsupported Roles**: Sending roles like `"assistant"` or `"system"` directly in the `contents` array. The Gemini API rejects the entire payload, returning a 400 Bad Request error.
- **Sending API Keys in Headers**: Gemini expects the API key to be sent as a URL query parameter `?key={API_KEY}` rather than in Authorization headers.

---

## 7. Visual Diagram (ASCII)
```
          [ Standard Client Messages ]
          - Role: system    -> maps to systemInstruction node
          - Role: user      -> maps to contents role: user
          - Role: assistant -> maps to contents role: model
                       |
                       v
         +----------------------------+
         |     Gemini JSON Payload    |
         +----------------------------+
                       |
                       v
    POST /models/{model}:streamGenerateContent?key=...
```

---

## 8. Interview Questions
1. **How does the Gemini API handle system prompts differently from OpenAI?**
   *Answer*: OpenAI passes system instructions as a message dictionary `{"role": "system", "content": "..."}` inside the message array. Gemini passes system instructions inside a separate, top-level `systemInstruction` parameter. This guarantees that instructions are parsed as core rules rather than conversation history.
2. **What roles are supported in the Gemini API `contents` block?**
   *Answer*: The Gemini API only supports `"user"` and `"model"` roles in the contents array. Any "system" instructions must be passed in the `systemInstruction` object, and "assistant" roles must be translated to "model" to avoid payload validation errors.

---

## 9. Key Takeaways
- Gemini requires roles to be strictly `"user"` or `"model"`.
- System instructions belong in the `systemInstruction` parameter.
- The API key is passed as a URL query parameter (`?key=...`).
- Streaming is implemented using `streamGenerateContent?alt=sse`.

---

## 10. Beginner Summary
Gemini Integration means connecting our chatbot directly to Google's AI brain. To do this, we structure our question inside a special JSON format that separates our strict instructions from our dialogue. We send the package directly to Google's server using our API key, which streams the answer back to our screen word by word.
