---
description: "Understand parameters, options, service dependencies and execution with Discussions' MessageSendCommand."
icon: terminal
---

# Commands


Commands organize text entry, parameter parsing and business calls. Discussions already has a MessageSendCommand, which is used to construct site messages and call MessageService; it is a real use case for the command class, but the current plugin manifest does not yet mount it to the terminal command tree.

## Options and Service Dependencies

Source: [src/Services/Commands/MessageSendCommand.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/Commands/MessageSendCommand.cs#L37) (excerpt; see source for context).

{% code title="MessageSendCommand.cs" %}
```csharp
[CommandOption(SUBJECT_OPTION, typeof(string), Required = true)]
[CommandOption(CONTENT_OPTION, typeof(string), Required = true)]
[CommandOption(CONTENTTYPE_OPTION, typeof(string))]
[CommandOption(MESSAGETYPE_OPTION, typeof(string))]
[CommandOption(SOURCE_OPTION, typeof(string))]
public class MessageSendCommand : CommandBase<CommandContext>
```
{% endcode %}
Source: [src/Services/Commands/MessageSendCommand.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/Commands/MessageSendCommand.cs#L58) (excerpt; see source for context).

{% code title="MessageSendCommand.cs" %}
```csharp
[ServiceDependency(Provider = Module.NAME)]
public MessageService Service { get; set; }
```
{% endcode %}

Subject and body are required, content type, message type and source are optional. The service dependency comes from the Discussions module, and you should not create another MessageService and accessor in the command.

## Construct Business Model at Execution Time

Source: [src/Services/Commands/MessageSendCommand.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/Commands/MessageSendCommand.cs#L63) (excerpt; see source for context).

{% code title="MessageSendCommand.cs" %}
```csharp
protected override ValueTask<object> OnExecuteAsync(CommandContext context, CancellationToken cancellation)
{
	if(context.Arguments == null || context.Arguments.IsEmpty)
		throw new CommandException("Missing arguments of the command.");

	var content = context.Options.GetValue<string>(CONTENT_OPTION);
	var contentType = context.Options.GetValue<string>(CONTENTTYPE_OPTION);

	//根据内容类型解析得到真实内容
	content = GetContent(content, ref contentType);

	var message = Zongsoft.Data.Model.Build<Models.Message>(entity =>
	{
		entity.Content = content;
		entity.ContentType = contentType;
		entity.Referer = context.Options.GetValue<string>(SOURCE_OPTION);
		entity.Subject = context.Options.GetValue<string>(SUBJECT_OPTION);
		entity.MessageType = context.Options.GetValue<string>(MESSAGETYPE_OPTION);
	});

	if(this.Service.Send(message, GetUsers(context.Arguments)) > 0)
		return ValueTask.FromResult<object>(message);

	return ValueTask.FromResult<object>(null);
}
```
{% endcode %}

Positional parameters resolve to recipient numbers, and options form the Message model. The method calls the on-site message service, not the message queue publisher. Return the model for upper layer processing; the command itself does not guarantee how the terminal formats the output.

## File Body Parameters

Source: [src/Services/Commands/MessageSendCommand.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/Commands/MessageSendCommand.cs#L91) (excerpt; see source for context).

{% code title="MessageSendCommand.cs" %}
```csharp
private static string GetContent(string content, ref string contentType)
{
	if(string.IsNullOrWhiteSpace(content) || string.IsNullOrWhiteSpace(contentType))
		return content;

	if(contentType.Length > 5 && contentType.EndsWith("+file", StringComparison.OrdinalIgnoreCase))
	{
		contentType = contentType.Substring(0, contentType.Length - 5);

		if(Zongsoft.IO.FileSystem.File.Exists(content))
		{
			using(var stream = Zongsoft.IO.FileSystem.File.Open(content, System.IO.FileMode.Open))
			{
				using(var reader = new System.IO.StreamReader(stream))
				{
					content = reader.ReadToEnd();
				}
			}
		}
	}

	return content;
}
```
{% endcode %}

A content type suffixed with +file means to read the body from the given path and remove this input tag. It is different from the +embedded convention used by persistent content and cannot be replaced by each other. The current logic retains the original value when the file does not exist. The caller needs to verify the input and cannot assume that the file content will always be read.

## From Class to Callable Command

For commands to enter the executor, the host also needs a command tree mount and the corresponding permission context. The documentation does not provide a command name that does not already exist in the repository. A complete assembly of existing terminal commands can be found in [Terminal](../terminals/commands.md) and the framework Main.plugin.
