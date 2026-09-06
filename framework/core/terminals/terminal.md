---
description: "Zongsoft.Terminals terminal abstraction, console terminal and interactive command mode."
icon: terminal
---

# Terminal

`Zongsoft.Terminals` abstracts the console program into a terminal and a terminal command executor. The terminal is responsible for input, output, style and interrupt events; the executor inherits the command execution model and is responsible for reading commands, executing commands and maintaining current command nodes.

`Terminal.Console` is the default console terminal provided by the core library, which encapsulates `System.Console` internally. `Terminal.Default` points to the console terminal by default and can be replaced by other implementations by the host or test code.

## Key Types

| Type | Description |
| --- | --- |
| `ITerminal` | Terminal interface, inherited from `ICommandOutlet`, provides input, output, error output, style, clear screen and reset. |
| `ITerminalExecutor` | Terminal command executor interface, inherits `ICommandExecutor`, provides `Run`/`RunAsync` and exit events. |
| `Terminal` | Static entry, providing `Default`, `Console`, output and style shortcut methods. |
| `ConsoleTerminal` | Console terminal implementation, encapsulating `System.Console`, ANSI style and Ctrl+C interrupts. |
| `ConsoleExecutor` | Console command executor, reads terminal input and executes commands. |
| `TerminalStyles` | Terminal style reset options. |
| `Terminal.ExitException` | Special exception used by terminal commands to exit the command loop. |

## Start Terminal

When running, the terminal executer prints a splash screen, enters a command loop, and waits for user input. You can pass in a plain text splash screen or a `CommandOutletContent` combined multi-color output.

Source: [framework/Zongsoft.Net/samples/server/Program.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Net/samples/server/Program.cs#L46) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
var splash = CommandOutletContent.Create()
	.AppendLine(CommandOutletColor.Yellow, new string('·', 50))
	.AppendLine(CommandOutletColor.Blue, "Welcome to the TCP Server.".Justify(50))
	.AppendLine(CommandOutletColor.Yellow, new string('·', 50));

await executor.RunAsync(splash);
```
{% endcode %}

The above is the startup fragment of the framework TCP server sample. The executor and start, stop, and info commands are created in the front of the same Program.cs; see [General Communication](../communication/general.md) for the complete entry. Discussions' own in-site messaging commands reuse the host executor and do not create repeated terminals.

The command loop will set the output encoding to UTF-8. The terminal style will be reset before each command is read, and a prompt will be displayed: `$>` will be displayed under the root node, and the full path of the node will be displayed after entering a command node.

## Command Loop

{% stepper %}
{% step %}
## Initialize Terminal

`ConsoleExecutor.RunAsync(...)` Set output encoding, print default or customized splash screen.
{% endstep %}

{% step %}
## Read Input

After the terminal is reset, the current command node prompt is displayed, and the command text is read from `Input.ReadLine()`.
{% endstep %}

{% step %}
## Execute Command

`ExecuteAsync(...)`, which inherits from `CommandExecutor`, is called when the input is non-null. If the command node has child nodes, the terminal executor will switch the current node to the command node, and subsequent relative paths will be searched using this node as the anchor point.
{% endstep %}

{% step %}
## Handling Exits and Exceptions

`ExitCommand` triggers an exit event after a terminal exit exception is thrown; ordinary exceptions will be output to the terminal with a red error message.
{% endstep %}
{% endstepper %}

## Interactive Response Mode

`Terminal.ReactiveAsync(...)` puts the command into "running and waiting for interrupt" mode. It will check whether the current executor is a terminal executor, mount the `Aborting` event of the terminal, then wait for Ctrl+C to release the semaphore, and finally execute the exit callback.

Source: [framework/Zongsoft.Commands/src/Messaging/QueueSubscribeCommand.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Commands/src/Messaging/QueueSubscribeCommand.cs#L52) (excerpt; see source for context).

{% code title="QueueSubscribeCommand.cs" %}
```csharp
protected override ValueTask<object> OnExecuteAsync(CommandContext context, CancellationToken cancellation) =>
	context.ReactiveAsync(this.OnEnterAsync, this.OnExitAsync, cancellation);
```
{% endcode %}

The message queue subscription command in `Zongsoft.Commands` has this structure: when entering, it subscribes to the queue and continuously outputs messages. After the user presses Ctrl+C, he exits and logs out of the subscription.

{% hint style="info" %}
`ReactiveAsync(...)` only supports terminal executor environments. Ordinary command executors do not have terminal interrupt events, and calling this method will throw an unsupported exception.
{% endhint %}

## Output Style

`ITerminal` inherits `ICommandOutlet`, so commands can use `CommandOutletContent` to combine colors, styles, and text fragments. The console terminal converts these patterns into ANSI escape sequence output.

Source: [framework/Zongsoft.Core/samples/memorycache/Program.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/samples/memorycache/Program.cs#L78) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
private static void Cache_Evicted(object sender, CacheEvictedEventArgs e)
{
	var content = CommandOutletContent.Create(CommandOutletColor.Magenta, "** Evicted **\t")
		.Append(CommandOutletColor.DarkGreen, Now + ' ')
		.Append(CommandOutletColor.Blue, $"[{e.Reason}] ")
		.Append(CommandOutletColor.DarkYellow, e.Key.ToString())
		.Append(CommandOutletColor.DarkGray, "=")
		.Append(CommandOutletColor.DarkYellow, e.Value?.ToString());

	if(e.State != null)
		content
			.Append(CommandOutletColor.DarkGray, " (")
			.Append(CommandOutletColor.Cyan, e.State.ToString())
			.Append(CommandOutletColor.DarkGray, ")");

	Terminal.WriteLine(content);
}
```
{% endcode %}

This output comes from Core's memorycache interaction sample, Now is a time text attribute of the same type, and e comes from the cache eviction event. `context.Output` is usually used first in commands, so that the same command can be output in the terminal and reused by other command executors. Only use `Terminal.WriteLine(...)` directly when you are sure that the output target is the current default terminal.

## Terminal Extension Methods

`Terminal.Utility` provides extension methods to get the terminal from the command executor or command context:

| method | Description |
| --- | --- |
| `GetTerminal(this ICommandExecutor executor)` | If the executor is a terminal executor, return the corresponding terminal. |
| `GetTerminal(this CommandContextBase context)` | Get the owning terminal from the command context. |
| `TryGetTerminal(...)` | Try to get a terminal, which is suitable for command-compatible terminal and non-terminal environments. |
| `ReactiveAsync(...)` | Puts the command into reactive mode exited by Ctrl+C. |

## Usage Suggestions

The terminal program is suitable for local debugging, plugin management, sample programs and operation and maintenance tools. For multi-user or remote management scenarios, you still need to consider permissions, auditing, command whitelists and external command restrictions.

If the command needs to run for a long time and wait for user interruption, give priority to using `ReactiveAsync(...)` to wrap the entry and exit logic; if the command only performs one action, directly implement ordinary `CommandBase<CommandContext>`.

## Reference Implementation

* [Terminals source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Terminals)
* [QueueSubscribeCommand.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Commands/src/Messaging/QueueSubscribeCommand.cs)
* [ZeroMQ terminal example](https://github.com/Zongsoft/framework/blob/main/messaging/zero/samples/server/Program.cs)
