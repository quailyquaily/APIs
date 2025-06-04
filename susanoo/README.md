# Susanoo APIs

**You need a susanoo API key to use the APIs.**

## Authentication

### Request Headers

| Header Name   | Value  | Description                                                        |
| ------------- | ------ | ------------------------------------------------------------------ |
| X-SUSANOO-KEY | SK-XXX | API key for authentication. Replace `SK-XXX` with your actual key. |

## 1. Create a Task

This API is used to create a task. The task includes a conversation with a series of messages and specific parameters for processing. The task will be processed by the backend system, and a unique trace ID will be returned for tracking the task's progress.

### Endpoint

```
POST /tasks
```

### Request Body

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Here is a fact, read it and say 'OK': Lucy has 3 apples, Lily has 2 more apples than Lucy.",
      "enable_cache": true
    },
    { "role": "assistant", "content": "OK." },
    { "role": "user", "content": "How many apples does Lily has?" }
  ],
  "params": {
    "format": "plaintext",
    "search": {
      "enabled": false,
      "limit": 10
    },
    "conditions": {
      "preferred_provider": "bedrock", // optional
      "preferred_model": "claude-3-5-sonnet" // optional
    }
  }
}
```

#### Request Parameters

| Parameter                            | Type    | Description                                                                                          |
| ------------------------------------ | ------- | ---------------------------------------------------------------------------------------------------- |
| messages                             | Array   | List of messages in the conversation. Each message includes a `role` (user/assistant) and `content`. |
| params                               | Object  | Additional parameters for the task.                                                                  |
| params.format                        | String  | Output format of the response (e.g., `plaintext`, `yaml` or `json`).                                 |
| params.search                        | Object  | Optional. Search parameters for the task.                                                            |
| params.search.enabled                | Boolean | Specifies whether to enable search for the task.                                                     |
| params.search.limit                  | Integer | Specifies the maximum number of search results to return.                                            |
| params.search.engine                 | String  | Specifies the search engine to use. Supported engine: `google`, `twitter`, `tavily`.                 |
| params.conditions                    | Object  | Optional. Conditions for task execution.                                                             |
| params.conditions.preferred_provider | String  | Specifies the preferred provider (e.g., `bedrock`, `azure`, `openai`, `deepseek`).                   |
| params.conditions.preferred_model    | String  | Specifies the model (e.g., `gpt-4o-mini`, `claude-3-5-sonnet`, `o1-mini`).                           |
| params.conditions.preferred_proxy    | String  | Specifies the proxy to use (e.g., `ada`, `cindy`).                                                   |

Supported Providers & Models:

| Provider     | Models                                                        |
| ------------ | ------------------------------------------------------------- |
| `bedrock`    | `claude-3-5-sonnet`                                           |
| `azure`      | `gpt-4o-mini`, `o1-mini`                                      |
| `openai`     | `gpt-4o-mini`, `o1-mini`, `o3`, `o3-mini`, `o4-mini`          |
| `anthropic`  | `claude-3-5-sonnet`                                           |
| `gemini`     | `gemini-2.5-pro`, `gemini-2.0-flash`, `gemini-2.0-flash-lite` |
| `xai`        | `grok-3`, `grok-3-mini`                                       |
| `deepseek`\* | `deepseek-chat`                                               |

\* Please contact us to enable these providers.

Output Format:

If `params.format` is set to `json` or `yaml`, susanoo will output the result in JSON or YAML format according to a template.

Which means you need to specify the output format in the `messages`, or provide a template in `params.format_template`.

For example:

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Lucy has 3 apples, Lily has 2 more apples than Lucy, how many apples does Lily have? output in JSON format. example: {\"apple_count\": NUMBER }"
    }
  ],
  "params": {
    "format": "json"
  }
}
```

or

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Lucy has 3 apples, Lily has 2 more apples than Lucy, how many apples does Lily have?"
    }
  ],
  "params": {
    "format": "json",
    "format_template": "{\"apple_count\": NUMBER }"
  }
}
```

### Response

#### Success Response

Status Code: `200 OK`

```json
{
  "data": {
    "code": 0,
    "trace_id": "7edde49c-bdd9-4998-b714-919df9929ec8"
  },
  "ts": 1736146770
}
```

#### Response Fields

| Field Name | Type    | Description                                                                            |
| ---------- | ------- | -------------------------------------------------------------------------------------- |
| code       | Integer | Indicates the status of the request (0 = success).                                     |
| trace_id   | String  | Unique identifier for the created task, which can be used to retrieve the task result. |
| ts         | Integer | Timestamp when the response was generated.                                             |

## 2. Retrieve Task Result

This API retrieves the result of a previously created task using the `trace_id`.

### Endpoint

```
GET /tasks/result?trace_id=:trace_id
```

### Query

| Parameter Name | Type   | Description                                                   |
| -------------- | ------ | ------------------------------------------------------------- |
| trace_id       | String | The unique identifier of the task to retrieve the result for. |

### Response

#### Success Response

Status Code: `200 OK`

```json
{
  "data": {
    "id": 9,
    "proxy_id": 2,
    "action": "",
    "messages": [
      {
        "content": "Here is a fact, read it and say 'OK': Lucy has 3 apples, Lily has 2 more apples than Lucy.",
        "role": "user"
      },
      {
        "content": "OK.",
        "role": "assistant"
      },
      {
        "content": "How many apples does Lily has?",
        "role": "user"
      }
    ],
    "params": {
      "format": "plaintext"
    },
    "result": {
      "response": {
        "json": null,
        "text": "To determine how many apples Lily has, we need to use the information provided:\n\n1. Lucy has 3 apples\n2. Lily has 2 more apples than Lucy\n\nWe can calculate this as follows:\nLily's apples = Lucy's apples + 2\nLily's apples = 3 + 2\nLily's apples = 5\n\nTherefore, Lily has 5 apples."
      }
    },
    "status": 3,
    "trace_id": "26929481-8461-4186-9f08-96d4308b6cfa",
    "scheduled_at": null,
    "created_at": "2025-01-02T10:30:27.358078+09:00",
    "updated_at": "2025-01-02T10:30:32.561192+09:00",
    "pending_count": 0
  },
  "ts": 1736146749
}
```

#### Response Fields

| Field Name   | Type    | Description                                                               |
| ------------ | ------- | ------------------------------------------------------------------------- |
| id           | Integer | Task ID.                                                                  |
| proxy_id     | Integer | Proxy ID for task execution.                                              |
| action       | String  | Action type for the task (if any).                                        |
| messages     | Array   | List of messages in the task.                                             |
| params       | Object  | Parameters used for the task.                                             |
| result       | Object  | The result of the task.                                                   |
| status       | Integer | Status of the task (1 = pending, 2 = running, 3 = completed, 4 = failed). |
| trace_id     | String  | Trace ID for the task.                                                    |
| scheduled_at | String  | Time when the task was scheduled (if any).                                |
| created_at   | String  | Time when the task was created.                                           |
| updated_at   | String  | Time when the task was last updated.                                      |
| ts           | Integer | Timestamp when the response was generated.                                |

#### Note

##### Output format

`params.format`'s value can be `plaintext`, `yaml` or `json`, but the `result` will always be a JSON object.

If `params.format` is set to `json`, the `result` will be the JSON response.

If `params.format` is set to `plaintext`, the `result` will be `{ "response": "THE_TEXT"}`

If `params.format` is set to `yaml`, the `result` will be `{ "yaml": "THE_YAML_CODE"}`

For example:

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Here is a fact, read it and say 'OK': Lucy has 3 apples, Lily has 2 more apples than Lucy."
    },
    { "role": "assistant", "content": "OK." },
    {
      "role": "user",
      "content": "How many apples does Lily has? please output in JSON format."
    }
  ],
  "params": {
    "format": "json",
    "format_template": "{\"apple_count\": NUMBER }"
  }
}
```

The response may be:

```json
{
  "data": {
    "id": 9,
    "proxy_id": 2,
    "action": "",
    "messages": [
      {
        "content": "Here is a fact, read it and say 'OK': Lucy has 3 apples, Lily has 2 more apples than Lucy.",
        "role": "user"
      },
      {
        "content": "OK.",
        "role": "assistant"
      },
      {
        "content": "How many apples does Lily has? please output in JSON format.",
        "role": "user"
      }
    ],
    "params": {
      "format": "json",
      "format_template": "{\"apple_count\": NUMBER }"
    },
    "result": { "apple_count": 5 },
    "status": 3,
    "trace_id": "26929481-8461-4186-9f08-96d4308b6cfa",
    "scheduled_at": null,
    "created_at": "2025-01-02T10:30:27.358078+09:00",
    "updated_at": "2025-01-02T10:30:32.561192+09:00",
    "pending_count": 0
  },
  "ts": 1736146749
}
```

##### Search Mode

For example:

```json
{
  "messages": [
    {"role": "user", "content": "What's Elon Musk's DOGE?"}
  ],
  "params": {
    {
      "search": {"limit": 20, "enabled": true, "engine": "google" },
      "conditions": {"preferred_model": "o1-mini"}
    }
  }
}
```

The response may be:

```json
{
  "data": {
    "id": 28,
    "proxy_id": 5,
    "action": "",
    "messages": [
      {
        "content": "What's Elon Musk's DOGE?",
        "role": "user"
      }
    ],
    "params": {
      "conditions": {
        "preferred_model": "o1-mini"
      },
      "search": {
        "enabled": true,
        "limit": 20
      }
    },
    "result": {
      "citations": [
        {
          "id": 1,
          "link": "https://www.forbes.com/sites/digital-assets/2024/11/14/elon-musk-issues-surprise-crypto-endorsement-amid-3-trillion-bitcoin-dogecoin-and-crypto-price-boom/",
          "snippet": "Nov 14, 2024 ... Elon Musk has praised the meme-based dogecoin, a cryptocurrency he once vowed to \"send to the moon\"...",
          "title": "Elon Musk Issues Surprise Crypto Endorsement Amid $3 Trillion ..."
        },
        {
          "id": 2,
          "link": "https://www.usatoday.com/story/money/2024/11/13/what-is-doge-elon-musk/76255384007/",
          "snippet": "Nov 13, 2024 ... Elon Musk has been tapped to co-lead the Department of Government Efficiency (DOGE) after years of hyping up cryptocurrency Dogecoin.",
          "title": "What is 'Doge'? Explaining the meme after Trump appoints Elon Musk"
        }
        // ...
      ],
      "refined_query": "What is Elon Musk's involvement with Dogecoin?",
      "response": "Elon Musk's **DOGE** primarily refers to the **Department of Government Efficiency**, a newly established advisory body appointed by former President Donald Trump {cite:2,12,14,19}. In this role, Musk, alongside entrepreneur Vivek Ramaswamy, is tasked with reducing government waste and enhancing operational efficiency across various government sectors {cite:2,5,12,14}. The acronym \"DOGE\" is a deliberate nod to **Dogecoin**, the meme-based cryptocurrency that Musk has long supported and promoted {cite:1,15,16}.\n\nThis dual use of \"DOGE\" underscores Musk's unique position at the intersection of technology, cryptocurrency, and governmental reform. While the Department of Government Efficiency aims to streamline government operations and reduce inefficiencies {cite:2,12,14,19}, the reference to Dogecoin highlights Musk's ongoing influence in the cryptocurrency market, where his endorsements have previously led to significant price movements and increased public interest {cite:1,15,16,19}.\n\nAdditionally, Musk's involvement with DOGE has sparked discussions about potential conflicts of interest and ethical considerations, given his prominent role in both the private sector (with companies like Tesla and SpaceX) and now a governmental advisory position {cite:13,20}. This multifaceted engagement demonstrates Musk's broad impact on both digital economies and public administration."
    },
    "status": 3,
    "trace_id": "425d8a15-d9b9-45ea-92eb-fbcb0fd4e0ef",
    "scheduled_at": null,
    "cost_time": 8130,
    "created_at": "2025-01-30T18:50:38.332091+09:00",
    "updated_at": "2025-02-05T13:43:46.584243+09:00",
    "pending_count": 0
  },
  "ts": 1738731061
}
```

in which

- `result.citations` contains the search results, each citation contains `id`, `link`, `snippet`, and `title`.
- `result.refined_query` contains the refined query used for search.

The citations are labeled with `{cite:ID}` or `{cite:ID1,ID2}` in the response text. You can use the `id` to refer to the citation in the response text.

## 3. Create Embedding

This API is used to create an embedding for a given text.

### Endpoint

```
POST /embeddings
```

### Request Body

```json
{
  "provider": "openai",
  "model": "text-embedding-3-small",
  "input": ["A beautiful sunset over the beach", "浜辺に沈む美しい夕日"],
  "openai_options": {
    // optional openai options
  },
  "jina_options": {
    // optional jina options
  }
}
```

### Request Parameters

| Parameter Name | Type   | Description                                        |
| -------------- | ------ | -------------------------------------------------- |
| provider       | String | Provider name.                                     |
| model          | String | Model name.                                        |
| input          | Array  | The text to create embedding for.                  |
| openai_options | Object | Optional openai options. will be passed to openai. |
| jina_options   | Object | Optional jina options. will be passed to jina.     |

Supported Providers & Models:

| Provider | Models                                         |
| -------- | ---------------------------------------------- |
| openai   | text-embedding-3-small, text-embedding-3-large |
| jina     | jina-embeddings-v3                             |

Additional Options:

You can pass additional options to the provider.

For example, for jina, you can specify the task adapter to better fit your use case (assume you are using jina for retrieval task).

```json
{
  "provider": "jina",
  "model": "jina-embeddings-v3",
  "input": ["What's the capital of France?"],
  "jina_options": {
    "task": "retrieval.query"
  }
}
```

Or for openai, you can specify the smaller dimensions for `text-embedding-3` and later models.

```json
{
  "provider": "openai",
  "model": "text-embedding-3-small",
  "input": ["A beautiful sunset over the beach", "浜辺に沈む美しい夕日"],
  "openai_options": {
    "dimensions": 128
  }
}
```

### Response

#### Success Response

Status Code: `200 OK`

```json
{
  "data": {
    "results": [
      {
        "embedding": "base64_encoded_embedding_string",
        "index": 1,
        "object": "embedding"
      },
      {
        "embedding": "base64_encoded_embedding_string",
        "index": 2,
        "object": "embedding"
      }
    ]
  },
  "ts": 1738731061
}
```

The `embedding` is base64 encoded. You need to decode the `embedding` to get the actual embedding vector, the array of floats.

For example:

```js
const decodedEmbedding = new Float32Array(
  Buffer.from(embedding, "base64").buffer
);
```

```python
decoded_embedding = np.frombuffer(
  base64.b64decode(embedding),
  dtype=np.float32
)
```

```go
decodedData, err := base64.StdEncoding.DecodeString(embedding)
if err != nil {
  return nil, err
}
const sizeOfFloat32 = 4
decodedEmbedding := make([]float32, len(decodedData)/sizeOfFloat32)
for i := 0; i < len(decodedEmbedding); i++ {
  decodedEmbedding[i] = math.Float32frombits(binary.LittleEndian.Uint32(decodedData[i*4 : (i+1)*4]))
}
fmt.Println(decodedEmbedding)
```

#### Error Response

```json
{
  "data": {
    "error": "..."
  },
  "ts": 1738731061
}
```

## 4. Rerank

This API is used to rerank a list of documents.

### Endpoint

```
POST /rerank
```

### Request Body

```json
{
  "model": "jina-reranker-v2-base-multilingual",
  "query": "Can I migrate from 微信公众号 to Quaily?",
  "top_n": 3,
  "documents": [
    {
      "text": "Once there is a little bird called quail, it will grow up to be a little girl."
    },
    {
      "text": "今日の天気はいいですね、明日も晴れです。一緒に外食しましょう。"
    },
    {
      "text": "During the California Gold Rush, some merchants made more money selling supplies to miners than the miners made finding gold."
    },
    {
      "text": "Quaily 可以从竹白等地方迁移过来。如果是公众号的话，需要确保版权没问题才行。"
    }
  ],
  "return_documents": false
}
```

### Request Parameters

| Parameter Name   | Type    | Description                                      |
| ---------------- | ------- | ------------------------------------------------ |
| provider         | String  | Provider name.                                   |
| model            | String  | Model name.                                      |
| query            | String  | The query to rerank the documents.               |
| top_n            | Integer | The number of documents to return.               |
| documents        | Array   | The list of documents to rerank.                 |
| return_documents | Boolean | Whether to return the documents in the response. |

Supported Models:

| Model Name                         | Description                    |
| ---------------------------------- | ------------------------------ |
| jina-reranker-m0                   | A multimodel reranker model.   |
| jina-reranker-v2-base-multilingual | A multilingual reranker model. |

### Response

#### Success Response

Status Code: `200 OK`

```json
{
  "data": {
    "results": [
      {
        "document": {},
        "index": 3,
        "relevance_score": 0.9453186827840592
      },
      {
        "document": {},
        "index": 0,
        "relevance_score": 0.40884310013171293
      },
      {
        "document": {},
        "index": 1,
        "relevance_score": 0.3465191827953349
      }
    ]
  },
  "ts": 1748469536
}
```

#### Error Response

```json
{
  "data": {
    "error": "..."
  },
  "ts": 1738731061
}
```

## 5. Classify

### Endpoint

```
POST /classify
```

### Request Body

```json
{
  "model": "jina-embeddings-v3",
  "input": [
    {
      "text": "A beautiful sunset over the beach"
    },
    {
      "text": "海滩上美丽的日落"
    },
    {
      "text": "浜辺に沈む美しい夕日"
    },
    {
      "text": "this apple is sweet! you should have a try!"
    },
    {
      "text": "this apple is sweet! bring some to Hawaii to share with your grandmother."
    },
    {
      "text": "Go to Hawaii to see your grandmother."
    }
  ],
  "labels": ["Food and drinks", "Nature and outdoors", "Traveling and Trips"]
}
```

### Request Parameters

| Parameter Name | Type   | Description                    |
| -------------- | ------ | ------------------------------ |
| model          | String | Model name.                    |
| input          | Array  | The list of texts to classify. |
| labels         | Array  | The list of labels.            |

Supported Models:

| Model Name         | Description                      |
| ------------------ | -------------------------------- |
| jina-embeddings-v3 | A multilingual embeddings model. |
| jina-clip-v2       | A multimodal embeddings model.   |

### Response

#### Success Response

Status Code: `200 OK`

```json
{
  "data": {
    "results": [
      {
        "index": 0,
        "object": "classification",
        "prediction": "Nature and Outdoors",
        "predictions": [
          {
            "label": "Food and Dining",
            "score": 0.3303683400154114
          },
          {
            "label": "Nature and Outdoors",
            "score": 0.3474631905555725
          },
          {
            "label": "Traveling and Trips",
            "score": 0.3221685290336609
          }
        ],
        "score": 0.3474631905555725
      }
      // ...
    ]
  }
}
```

#### Error Response

```json
{
  "data": {
    "error": "..."
  }
}
```

## 6. Generate Images

### Endpoint

```
POST /images/generations?async=0
```

### Request Body

```json
{
  "model": "gpt-image-1",
  "prompt": "A rabbit is eating a tiger"
}
```

### Request Parameters

| Parameter Name | Type    | Description                       |
| -------------- | ------- | --------------------------------- |
| provider       | String  | Provider name.                    |
| model          | String  | Model name.                       |
| prompt         | String  | The prompt to generate the image. |
| n              | Integer | The number of images to generate. |
| openai_options | Object  | optional, OpenAI options.         |
| gemini_options | Object  | optional, Gemini options.         |

Supported Models:

| Model Name  | Description                       |
| ----------- | --------------------------------- |
| gpt-image-1 | A text-to-image generation model. |

### Query Parameters

| Parameter Name | Type    | Description       |
| -------------- | ------- | ----------------- |
| async          | Integer | optional, 0 or 1. |

- `async=0`: The API will return the image.
- `async=1`: The API will return the task id.

### Response

#### Success Response

Status Code: `200 OK`

For `async=0`:

```json
{
  "data": {
    "expires": "2025-06-04T04:26:24Z",
    "results": [
      {
        "b64_json": "base64_encoded_image_string"
      },
      {
        // ...
      }
    ]
  },
  "ts": 1738731061
}
```

For `async=1`:

```json
{
  "data": {
    "code": 0,
    "trace_id": "UUID"
  },
  "ts": 1749007509
}
```

_Note: The `b64_json` field is base64 encoded image string, and the generated image will expire after 20 minutes._

#### Decode the image

```go
imgData, err := base64.StdEncoding.DecodeString(Base64Encoded)
if err != nil {
  fmt.Errorf("failed to decode image data: %w", err)
  return
}

imgObj, err := png.Decode(bytes.NewBuffer(imgData))
if err != nil {
  fmt.Errorf("failed to decode image: %w", err)
  return
}
```
