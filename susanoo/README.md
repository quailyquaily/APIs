# Susanoo APIs

**You need a susanoo API key to use the APIs.**

## 1. Create a Task

### Endpoint

```
POST /tasks
```

### Description

This API is used to create a task. The task includes a conversation with a series of messages and specific parameters for processing. The task will be processed by the backend system, and a unique trace ID will be returned for tracking the task's progress.

### Request Headers

| Header Name   | Value            | Description                                                       |
| ------------- | ---------------- | ----------------------------------------------------------------- |
| Content-Type  | application/json | Specifies the media type of the request body.                     |
| X-SUSANOO-KEY | SK-XXX           | API key for authentication. Replace `SK-XX` with your actual key. |

### Request Body

```json
{
  "messages": [
    {
      "role": "user",
      "content": "Here is a fact, read it and say 'OK': Lucy has 3 apples, Lily has 2 more apples than Lucy."
    },
    { "role": "assistant", "content": "OK." },
    { "role": "user", "content": "How many apples does Lily has?" }
  ],
  "params": {
    "format": "plaintext",
    "conditions": {
      "preferred_provider": "bedrock", // optional
      "preferred_model": "claude-3-5-sonnet" // optional
    }
  }
}
```

#### Request Parameters

| Parameter                            | Type   | Description                                                                                          |
| ------------------------------------ | ------ | ---------------------------------------------------------------------------------------------------- |
| messages                             | Array  | List of messages in the conversation. Each message includes a `role` (user/assistant) and `content`. |
| params                               | Object | Additional parameters for the task.                                                                  |
| params.format                        | String | Output format of the response (e.g., `plaintext` or `json`).                                         |
| params.conditions                    | Object | Conditions for task execution.                                                                       |
| params.conditions.preferred_provider | String | Specifies the preferred provider (e.g., `bedrock`, `azure`).                                         |
| params.conditions.preferred_model    | String | Specifies the model (e.g., `4o-mini`, `claude-3-5-sonnet`, `claude-3-5-sonnet-v2`).                  |

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

### Endpoint

```
GET /tasks/result?trace_id=:trace_id
```

### Description

This API retrieves the result of a previously created task using the `trace_id`.

### Request Headers

| Header Name   | Value  | Description                                                        |
| ------------- | ------ | ------------------------------------------------------------------ |
| X-SUSANOO-KEY | SK-XXX | API key for authentication. Replace `SK-XXX` with your actual key. |

### Request Query Parameters

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
        "Json": null,
        "Text": "To determine how many apples Lily has, we need to use the information provided:\n\n1. Lucy has 3 apples\n2. Lily has 2 more apples than Lucy\n\nWe can calculate this as follows:\nLily's apples = Lucy's apples + 2\nLily's apples = 3 + 2\nLily's apples = 5\n\nTherefore, Lily has 5 apples."
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

| Field Name      | Type    | Description                                                               |
| --------------- | ------- | ------------------------------------------------------------------------- |
| id              | Integer | Task ID.                                                                  |
| proxy_id        | Integer | Proxy ID for task execution.                                              |
| action          | String  | Action type for the task (if any).                                        |
| messages        | Array   | List of messages in the task.                                             |
| params          | Object  | Parameters used for the task.                                             |
| result          | Object  | The result of the task.                                                   |
| result.response | Object  | Contains the result in `Json` and `Text`.                                 |
| status          | Integer | Status of the task (1 = pending, 2 = running, 3 = completed, 4 = failed). |
| trace_id        | String  | Trace ID for the task.                                                    |
| scheduled_at    | String  | Time when the task was scheduled (if any).                                |
| created_at      | String  | Time when the task was created.                                           |
| updated_at      | String  | Time when the task was last updated.                                      |
| ts              | Integer | Timestamp when the response was generated.                                |

#### Note

`params.format`'s value can be `plaintext` or `json`, but the `result` will always be a JSON object.

If `params.format` is set to `json`, the `result` will be the JSON response. If `params.format` is set to `plaintext`, the `result` will be `{ "response": "THE_TEXT"}`

If you prefer to ask LLM to output the result in JSON format, you can set `params.format` to `json`, and ask they to output the result in JSON format in the `messages`.

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
      "content": "How many apples does Lily has? please output in JSON format. example: {\"apple_count\": NUMBER }"
    }
  ],
  "params": {
    "format": "json"
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
        "content": "How many apples does Lily has? please output in JSON format. example: {\"apple_count\": NUMBER }",
        "role": "user"
      }
    ],
    "params": {
      "format": "json"
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