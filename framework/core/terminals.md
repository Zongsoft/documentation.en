---
description: "Zongsoft.Terminals terminal abstraction, command executor and interactive console program."
icon: terminal
---

# Zongsoft.Terminals

`Zongsoft.Terminals` provides terminal abstraction, console terminal implementation and terminal command executor, which are used to run the `Zongsoft.Components` command model in the interactive console. It allows a program to start components like a normal background service, and can also enter commands in the console, view status, perform maintenance actions, and enter continuous monitoring mode.

The core relationship of the terminal model is: `ITerminal` is responsible for input, output, style, screen clearing and Ctrl+C interrupt events; `ITerminalExecutor` inherits the command executor and is responsible for reading command text, parsing command expressions, maintaining the current command node and triggering exit events.

## Main Responsibilities

* Define terminal abstractions such as `ITerminal`, `ITerminalExecutor`, etc.
* Provides `Terminal.Console` console terminal and `Terminal.Default` default terminal entrance.
* Combine command trees, command expressions, and terminal input loops into interactive command line programs.
* Supports color output, ANSI style, screen clear, reset and Ctrl+C interrupt handling.
* Provides built-in console commands such as `Clear`, `Exit`, `Shell`, etc.
* Support the plugin host to replace the default command executor with the terminal executor, so that plugin commands can be run in the terminal.

## Key Types

| Type | Description |
| --- | --- |
| `ITerminal` | Terminal interface provides input and output, error output, style, screen clear, reset and interrupt events. |
| `ITerminalExecutor` | Terminal command executor interface, inherits the command executor and provides `Run(...)` / `RunAsync(...)`. |
| `Terminal` | Static entry, exposing `Console`, `Default`, `Write(...)`, `WriteLine(...)`, `Clear(...)` and other shortcut methods. |
| `TerminalStyles` | Terminal style reset options such as foreground color, background color, and font style. |
| `Terminal.ExitException` | Special exception used to terminate the terminal command loop. |
| `Terminal.ExitEventArgs` | Terminal exit event parameters, including exit code. |

## Applicable Scenarios

* Provides interactive command entry for local debugging, operation and maintenance tools, and sample programs.
* Maintenance commands such as `start`, `stop`, `info`, `subscribe` are provided next to the background service.
* Hang the commands in the plugin tree to the terminal to form an extensible management console.
* Provides a "run until Ctrl+C exits" response mode for scenarios such as message subscription, event monitoring, and device collection and observation.

## Quick Start

The actual command of Discussions is MessageSendCommand: it builds the in-site message from the command options, parses the recipient and hands it to the MessageService. It does not create its own console loop, the host provides execution entry through the command tree and plugin assembly.

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

The terminal executor comes with the `Exit`, `Clear`, and `Shell` commands by default. Business commands can be mounted through the command tree, plugin loader, or auxiliary methods such as `CommandExecutorUtility.Command(...)`. For the complete command model, see:

{% content-ref url="components/commands.md" %}
[commands.md](components/commands.md)
{% endcontent-ref %}

## Plugin Hosting Integration

In the plugin host, the terminal plugin will point `/Workbench/Executor` to `Terminal.Console.Executor` and set it to `CommandExecutor.Default`. In this way, commands mounted to the command tree by other plugins can be directly entered and executed in the terminal host.

Source: [framework/Zongsoft.Plugins/plugins/Terminal.plugin](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/plugins/Terminal.plugin#L15) (excerpt; see source for context).

{% code title="Terminal.plugin" %}
```xml
<extension path="/Workbench">
	<!-- 将默认的命令执行器替换成控制台终端命令执行器 -->
	<object name="Executor" value="{static:Zongsoft.Terminals.Terminal.Console.Executor, Zongsoft.Core}">
		<object.property name="Default" target="{type:Zongsoft.Components.CommandExecutor, Zongsoft.Core}" value="{path:.}" />
	</object>
</extension>
```
{% endcode %}

This method is suitable for turning plugin applications into interactive management consoles. After the application starts, the terminal [workbench](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/src/IWorkbenchBase.cs) runs the executor; when the terminal exits, the workbench closes.

## Sub Namespace

| namespace | Description |
| --- | --- |
| `Zongsoft.Terminals.Commands` | Terminal built-in commands. |

## Type

<table data-view="cards">
	<thead>
		<tr>
			<th></th>
			<th data-card-target data-type="content-ref"></th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>Terminal: Terminal abstraction, console terminal, terminal executor, output styles and reactive commands.</td>
			<td><a href="terminals/terminal.md">terminal.md</a></td>
		</tr>
		<tr>
			<td>Commands: Clear screen, exit and Shell commands.</td>
			<td><a href="terminals/commands.md">commands.md</a></td>
		</tr>
	</tbody>
</table>

## Related Resources

* [Terminals source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Terminals)
* [Terminal plugin](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Plugins/plugins/Terminal.plugin)
