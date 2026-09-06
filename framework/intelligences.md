---
description: "Configure the AI assistant, connect to model services, and use chat capabilities through the terminal or the web."
icon: brain
---

# AI Integration

`Zongsoft.Intelligences` organizes model service integration into helpers, models, and chat services. It is suitable for plugin applications to use a unified entrance calling model, and provides terminal commands for operation and maintenance personnel and a Web interface for clients. Model inference is still performed by the connected model service; deploying the plugin does not also install the model server or download the model.

## First Understand the Four Concepts

| concept | Responsibilities | Choices when configuring or using |
| --- | --- | --- |
| provider | Adaptation model service agreement | For example `ollama` driver |
| Assistant | Name a set of connection settings and service capabilities | For example, an assistant named `ollama` |
| model | The model that actually performs inference | `model` in connection, must be available on the server |
| session | Save the status and history of ongoing conversations | Contextual multi-round chat; see [Sessions and Streaming Responses](intelligences/sessions.md) for details |

The assistant name and driver name can be the same, but they are not the same concept. The same driver can be configured with multiple helpers for different servers or models. `AssistantManager` is the entrance to the search assistant; the specific assistant provides capabilities through chat services and model services. Callers should not treat server-specific model management operations as functionality supported by all providers.

## Integration Local Model Service

First prepare accessible Ollama services and installed models. Discussions There is no model calling use case; the framework AI Integration plugin package option is used below, where qwen3:0.6b is the original model configuration and should be replaced with the model name actually provided by the server. Model license, memory requirements, context length, and tool call support are determined by the selected model and service.

Add the plugin to the host's deployment manifest; the terminal host must have an existing terminal command infrastructure.

{% code title=".deploy" %}
```ini
[plugins zongsoft intelligences]
nuget:Zongsoft.Intelligences
```
{% endcode %}

Merge settings into the application's own `.option` file; do not rely on overwriting in-package files to save environment configuration.

Source: [framework/Zongsoft.Intelligences/src/Zongsoft.Intelligences.option](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Intelligences/src/Zongsoft.Intelligences.option#L3) (excerpt; see source for context).

{% code title="Zongsoft.Intelligences.option" %}
```xml
<options>
	<option path="ai">
		<connectionSettings>
			<connectionSetting connectionSetting.name="ollama"
			                   driver="ollama"
			                   value="server=http://127.0.0.1:11434;model=qwen3:0.6b" />
		</connectionSettings>
	</option>
</options>
```
{% endcode %}

Replace the file name with the host's actual application name according to [Options file matching rules](../references/option-files.md). `127.0.0.1` in the container points to the container itself, and the model server should be changed to a reachable address when it is in other containers or hosts.

## Verify from Terminal

The following command chain framework has terminal commands as steps; it is not a statement that Discussions has deployed the AI assistant. First confirm that the assistant is found, then confirm that the model service is available, and finally initiate a short conversation. This distinguishes between configuration loading failures and remote inference failures.

{% code title="Assistant.commands" %}
```text
ai.assistant
ai.assistant ollama
ai.assistant.model.list
ai.assistant.chat.open
ai.assistant.chat "Introduce yourself in one sentence." --format:text --streaming
ai.assistant.chat.history
ai.assistant.chat.close
```
{% endcode %}

`ai.assistant ollama` selects the current assistant; the chat subcommand works in that context. `model.list --running` checks the running model, and `model.info` checks the specified model information. `model.install` and `model.uninstall` will change the model server status. Use the command help to confirm the parameters and target assistant before use.

💡 Use a short question to complete the verification first, and then introduce long prompt words, tool calls and multiple rounds of history. Long periods of unresponsiveness may be caused by a cold start of the model, insufficient resources, or network problems, not necessarily by the plugin not being loaded.

## Provide Web Interface

Deploy additional `Zongsoft.Intelligences.Web` on the existing [Web Host](../hosting/web.md). The entrance to the assistant list is `/AI/Assistants`, and the entrance to the model is `/AI/Assistants/{name}/Models`; see [conversational article](intelligences/sessions.md) for the session, history and chat interfaces.

The interface layer does not replace the business's own user authentication, assistant access scope, request quota, and session ownership verification. In particular, model installation, deletion and session enumeration capabilities should generally only be open to users with corresponding permissions.

## Prompt Words and Ability Boundaries

The default implementation loads preset prompt words matching the assistant name from the `preludes` directory next to the assembly. Preset prompt words are application configuration and should not be overridden by unchecked user input. It can define answer formats and task context, but it cannot replace permission checks or limit the actual capabilities of the called tool.

Currently this module mainly provides assistants, models and chat infrastructure. Search enhancement generation also requires the application to organize its own data segmentation, indexing, retrieval and citation; automatic execution tools also need to clarify tool permissions and error handling. Don't assume that a complete knowledge base or autonomous task system is in place just because an interface can call a model.

## Check Order

1. The assistant list is empty: check whether the `.option` file matches the application name and whether the plugin enters the scan directory.
2. Helper exists but model list fails: Check `server`, container network, service listening address and protocol.
3. The model list is normal but the chat fails: check the model name, model loading status and server logs.
4. Single round normal, multiple rounds abnormal: check session ID, expiration, history size and concurrent requests, continue reading [Sessions and Streaming Responses](intelligences/sessions.md).

Source references: [Assistant and chat implementation](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Intelligences/src), [Web controller](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Intelligences/api/Controllers).
