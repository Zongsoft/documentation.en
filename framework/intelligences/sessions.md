---
description: "Understand chat history, streaming enumeration, session isolation, and post-interruption state."
icon: comments
---

# Sessions and Streaming Responses

The chat model typically generates answers based on the context of this submission. What the application calls "remembering the last conversation" may be to resend the history each time, or it may be that the model service provides a session ID and saves the context. The resource consumption and status ownership of the two are different.

## History of How to Participate in Requests

Currently, `ChatSession` will add user messages to the local history before requesting the chat service. When there is no server `ConversationId`, the request includes the preset prompt word and existing history; when there is this identifier, the request uses the preset prompt word and the current message, and the continuous context is handed over to the corresponding provider for processing.

Therefore, local history does not equal server-side state, nor does it equal complete audit records. Whether clearing the local history also clears the context saved by the server cannot be judged based only on the changes in the local collection.

{% hint style="warning" %}
🚨 User messages may have been written to history before the request failed. When retrying the business, you should decide to reuse the session, clean up the failure message, or create a new session to avoid continuously appending the same problems to the history.
{% endhint %}

## Normal Response and Streaming Response

A normal response adds the return message to the history after getting the result. The streaming response returns updates item by item, and adds the accumulated helper text to the history after complete enumeration of the stream.

| scene | What the caller has to do |
| --- | --- |
| Full read response | Display the incremental content and finally record the completion status |
| User stops generating | Cancel the upstream call and clearly show that the answer is not completed |
| Connection interrupted | Keep the displayed text and error status, and do not pretend to be a complete answer |
| Audit tool calls are required | Save structured events separately; don’t rely solely on streaming cumulative text |

The streaming response is an asynchronous data sequence; getting the sequence does not mean that the request has been completed successfully. Exceptions may occur during subsequent enumerations. The read loop should be wrapped in error handling and cancellation, not just the line that gets the sequence.

## Session Isolation and Lifecycle

`Current` for a session manager represents the current session of that manager, not the context of an automatically bound HTTP user. Web applications should pass the session ID explicitly and maintain the user or tenant → session allowed access relationship. Random identification can only reduce the chance of guessing and cannot replace attribution verification.

Concurrent requests in the same session will share history and options; the current call may also modify `ChatOptions.ConversationId`. It is recommended to serialize single-session requests, or to create independent session and options objects for independent tasks. Do not reuse the same mutable options instance arbitrarily for multiple concurrent requests.

`Abandon` will release the session. The release of the current session also involves the release of the chat service. Therefore, if a custom implementation shares the same service instance between multiple sessions, it must check resource ownership to avoid closing one session and affecting other sessions. Session lifetime and in-process history also cannot be used as persistence promises.

{% hint style="info" %}
💡 The current session enumeration implementation depends on the target framework:.NET 9 and above can enumerate cache keys, and the .NET 8 path will throw an unsupported exception. Before integrating the session list interface, you should check the actual deployment target, not just the list of frameworks supported by the package.
{% endhint %}

## Web Call Path

The following path takes the assistant named `ollama` as an example. See [AI Integration](../intelligences.md) for deployment and connection settings.

| methods and paths | meaning |
| --- | --- |
| `POST /AI/Assistants/ollama/Chats` | Create a session and return the session ID |
| `GET /AI/Assistants/ollama/Chats/{id}` | Read session summary |
| `POST /AI/Assistants/ollama/Chats/{id}/Chat` | Chat in the specified session, the request body is text |
| `POST /AI/Assistants/ollama/Chats/Chat` | Chat without local conversation history |
| `GET /AI/Assistants/ollama/Chats/{id}/History` | Get character and text history |
| `DELETE /AI/Assistants/ollama/Chats/{id}/History` | Clear local history |
| `DELETE /AI/Assistants/ollama/Chats/{id}` | Remove and release session |

The current chat controller will return a no-session call when the specified identifier is not found, instead of uniformly returning that the session does not exist. Businesses that require strict continuity should first verify the existence of the session and verify ownership. Do not regard "received reply" as evidence that "old history has participated in the request".

HTTP responses are output by the Web's asynchronous enumeration response mechanism and should not be relied upon as a vendor's SSE protocol without verification. The client should parse according to the actual content type and response format, and verify mid-cancellation, proxy buffering, timeout, and disconnection reconnection behaviors.

## Acceptance Scenario During Integration

First verify two rounds of questions and answers in the same session, and then test that the sessions of the two users cannot access each other; then test the cancellation during generation, server-side error reporting, expiration identification, and concurrent submission. If cross-process recovery is required, the necessary historical and business metadata should be persisted, and it should be clear how to reconstruct the context after recovery.

Source references: [ChatSession](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Intelligences/src/ChatSession.cs), [Session web interface](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Intelligences/api/Controllers/ChatController.cs).
