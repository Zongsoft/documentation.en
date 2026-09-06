---
description: "Zongsoft.Terminals.Commands Terminal built-in commands."
icon: keyboard
---

# Commands

`Zongsoft.Terminals.Commands` provides built-in commands that are loaded by default in the console terminal. These commands will be added to the root command node when `ConsoleExecutor` is constructed, so they can be used after the terminal program is started.

These commands are designed around the terminal environment. `Exit` and `Shell` need to obtain the current terminal from the command context; if run directly in a normal command executor, an unsupported exception will usually be thrown or have no practical effect.

## Command List

| Commands | Type | Description |
| --- | --- | --- |
| `Clear` | `ClearCommand` | Call `Clear()` of the current terminal to clear the screen. |
| `Exit` | `ExitCommand` | Exit the terminal command loop and support `--yes` or `-y` to skip confirmation. |
| `Shell` | `ShellCommand` | Execute an external command via `cmd.exe /C` on Windows and write standard output back to the terminal. |

## ClearCommand

`ClearCommand` will attempt to clean up the console output using the current terminal's screen clearing capabilities. The console terminal will first write the ANSI screen clear sequence and then call `System.Console.Clear()`.

{% code title="clear screen" %}
```bash
Clear
```
{% endcode %}

If the command is not run in a terminal executor, `ClearCommand` will not clear the screen and will not output additional results when the terminal cannot be obtained.

## ExitCommand

`ExitCommand` only supports running in a terminal executor. When `--yes` is not specified, a confirmation prompt will be output to the terminal. After the user enters `yes`, a terminal exit exception will be thrown, and `ConsoleExecutor` will capture and trigger the exit event.

{% code title="Exit terminal" %}
```bash
Exit --yes
```
{% endcode %}

Short options are also available:

{% code title="Use short option to exit" %}
```bash
Exit -y
```
{% endcode %}

If `--yes` is not specified, the command reads terminal input; it will only exit if `yes` is entered. Other input returns the command loop.

## ShellCommand

`ShellCommand` is used to temporarily execute system commands in the terminal. It supports the `timeout` option and the default value is `1s`. The current implementation is for the Windows console and will execute the first command parameter through `cmd.exe /C`; non-Windows platforms will throw an unsupported exception.

{% code title="Execute shell command" %}
```bash
Shell "dotnet --info" --timeout:10s
```
{% endcode %}

If the process does not exit within the timeout period, the command returns `-1`; otherwise, the external process exit code is returned. The current implementation uses a 30 second wait time when `timeout` is less than or equal to zero.

{% hint style="warning" %}
`ShellCommand` will execute an external system command. This command should be opened with caution in a production environment or multi-user terminal. A safer approach is to only mount clear business commands rather than exposing arbitrary shell capabilities to end users.
{% endhint %}

## Command Resources

Display names, descriptions, and option descriptions for built-in commands come from resource files, such as `ExitCommand.Name`, `ExitCommand.Description`, `ShellCommand.Options.Timeout`. If the terminal program requires multi-language display, it is recommended to prioritize maintaining resources instead of hard-coding display text in the command logic.

## Usage Suggestions

Built-in commands are suitable for local terminals and development tools. Terminals released to the production environment or remote users should decide whether to retain `Shell` based on permissions and operation and maintenance policies. The command tree can also be adjusted at startup to expose only the command set allowed by the business.

## Reference Implementation

* [ClearCommand.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Terminals/Commands/ClearCommand.cs)
* [ExitCommand.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Terminals/Commands/ExitCommand.cs)
* [ShellCommand.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Terminals/Commands/ShellCommand.cs)
