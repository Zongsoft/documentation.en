---
description: "ITransmitter, TransmitterDescriptor, TransmitterHandler and template notification sending."
icon: paper-plane
---

# Transmitter

`Transmitter` is a templated messaging model. It breaks down "who sends it, which channel to send it through, which template to use, what parameters the template requires, and who to send it to" into a descriptive, configurable, and executable structure that is suitable for SMS, voice, WeChat template messages, verification codes, and business notifications.

## Type Relationship

| Type | Description |
| --- | --- |
| `ITransmitter` | Template message sender interface. |
| `ITransmitterArgumenter` | Convert the handler parameter set to a template parameter object. |
| `TransmitterDescriptor` | Sender metadata describing name, title, channel, and template. |
| `TransmitterDescriptor.Channel` | Sending channels, such as SMS, voice, and WeChat template messages. |
| `TransmitterDescriptor.Template` | Template definition. |
| `TransmitterDescriptor.Template.Parameter` | Template parameter definition. |
| `TransmitterHandler` | handler entry, find the sender according to parameters and call sending. |
| `TransmitterUtility` | Chained extension methods for building descriptors. |

## Why Do You Need Descriptor?

`TransmitterDescriptor` is not just a description. It lets the backend or GUI dynamically know:

* What transmitters are there in the system, such as `Phone`, `Wechat`.
* What channels does each transmitter have, for example `message`, `voice`.
* What templates are there for each channel.
* What parameters are required for each template.

This is critical for users to define notification rules on the interface. After the interface can read the descriptor, it generates the channel drop-down box, template drop-down box, and parameter input form, and then saves the user configuration as executable `TransmitterHandler.Argument`.

## Build Descriptor

Discussions There are currently no template SMS or voice sending use cases. The Aliyun PhoneTransmitter of the framework reads text messages and voice templates from the real option set and creates descriptors. The template identifier and parameters come from the configured service, and no additional recipients or alarm templates are created in the document.

Source: [framework/externals/aliyun/src/Telecom/PhoneTransmitter.cs](https://github.com/Zongsoft/framework/blob/main/externals/aliyun/src/Telecom/PhoneTransmitter.cs#L65) (excerpt; see source for context).

{% code title="PhoneTransmitter.cs" %}
```csharp
public TransmitterDescriptor Descriptor
{
	get
	{
		if(_descriptor == null)
		{
			_descriptor = new TransmitterDescriptor(this.Name, AnnotationUtility.GetDisplayName(this.GetType()), AnnotationUtility.GetDescription(this.GetType()));

			var channel = _descriptor.Channel(MESSAGE_CHANNEL, Properties.Resources.Text_Phone_Message);
			foreach(var option in this.Phone.Options.Message.Templates)
			{
				var template = channel.Template(option.Name);

				foreach(var parameter in option.Parameters)
					template.Parameter(parameter.Name, parameter.Title, parameter.Description);
			}

			channel = _descriptor.Channel(VOICE_CHANNEL, Properties.Resources.Text_Phone_Voice);
			foreach(var option in this.Phone.Options.Voice.Templates)
			{
				var template = channel.Template(option.Name);

				foreach(var parameter in option.Parameters)
					template.Parameter(parameter.Name, parameter.Title, parameter.Description);
			}
		}

		return _descriptor;
	}
}
```
{% endcode %}

`TransmitterUtility` provides `Channel`, `Template`, and `Parameter` extension methods to facilitate the sender to construct descriptors from configuration or remote services at runtime.

## Execute Send

The core method of `ITransmitter` is `TransmitAsync`. `destination` is the destination, such as mobile phone number, OpenId, email or custom address; `channel` is the channel; `template` is the template identifier; `data` is the template parameter object. The following is the actual call fragment of the framework Secretor to generate the confirmation code and delegate sender delivery after completing the scheme and verification code verification; its variables come from the outer TransmitAsync method and cannot be directly exposed as a public sending interface without the verification step.

Source: [framework/Zongsoft.Core/src/Security/Secretor.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Secretor.cs#L355) (excerpt; see source for context).

{% code title="Secretor.cs" %}
```csharp
var token = GetKey(scheme, destination, template, scenario, channel);
var value = await _secretor.GenerateAsync(token, null, extra, cancellation);

//发送验证码到目的地
await transmitter.TransmitAsync(destination, channel, template, new SecretTemplateData(value), cancellation);
```
{% endcode %}

`TransmitterHandler` wraps the sending action into a general handler. It will search for `ITransmitter` by `Argument.Name`, then send by `Argument.Channel`, `Argument.Template` and `Argument.Destination`.

Source: [framework/Zongsoft.Core/src/Communication/TransmitterHandler.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Communication/TransmitterHandler.cs#L49) (excerpt; see source for context).

{% code title="TransmitterHandler.cs" %}
```csharp
protected override ValueTask OnHandleAsync(Argument argument, Collections.Parameters parameters, CancellationToken cancellation)
{
	if(argument == null)
		return ValueTask.CompletedTask;

	//获取指定名称的发送器
	var transmitter = _serviceProvider.FindRequired<ITransmitter>(argument.Name);

	//获取指定的发送通道
	if(!transmitter.Descriptor.Channels.TryGetValue(argument.Channel ?? string.Empty, out var channel))
		channel = transmitter.Descriptor.Channels.Count > 0 ? transmitter.Descriptor.Channels[0] : null;

	//如果没有指定参数对象则尝试通过参数转换器将参数集转换成发送模板的参数对象
	if(argument.Parameter == null)
	{
		//获取指定的发送器参数转换器
		var argumenter = _serviceProvider.Find<ITransmitterArgumenter>(argument);

		if(argumenter != null)
			argument.Parameter = argumenter.GetArgument(transmitter, channel?.Name ?? argument.Channel, argument.Template, argument.Parameter, parameters);
		else
			argument.Parameter = parameters;
	}

	//执行发送任务
	return transmitter.TransmitAsync(argument.Destination, argument.Channel, argument.Template, argument.Parameter, cancellation);
}
```
{% endcode %}

If `Argument.Parameter` is empty, `TransmitterHandler` will try to find `ITransmitterArgumenter` from the service container and convert the current handler parameter set into a template parameter object. This allows the GUI to just save the "template parameter mapping rules" and then have the parameter converter generate the final object at runtime.

## Aliyun PhoneTransmitter

`PhoneTransmitter` in `externals/aliyun` is the most typical implementation:

* `Name` is fixed to `Phone`.
* `Descriptor` generates `message` channels based on `Phone.Options.Message.Templates`.
* `Descriptor` generates `voice` channels based on `Phone.Options.Voice.Templates`.
* In `TransmitAsync`, the `message` channel calls `Phone.SendAsync`, and the `voice` channel calls `Phone.CallAsync`.

Source: [framework/externals/aliyun/src/Telecom/PhoneTransmitter.cs](https://github.com/Zongsoft/framework/blob/main/externals/aliyun/src/Telecom/PhoneTransmitter.cs#L98) (excerpt; see source for context).

{% code title="PhoneTransmitter.cs" %}
```csharp
public async ValueTask TransmitAsync(string destination, string channel, string template, object data, CancellationToken cancellation)
{
	if(data == null)
		throw new ArgumentNullException(nameof(data));

	if(string.IsNullOrEmpty(channel) || string.Equals(channel, MESSAGE_CHANNEL, StringComparison.OrdinalIgnoreCase))
		await this.Phone.SendAsync(template, new[] { destination }, data, cancellation:cancellation);
	else if(string.Equals(channel, VOICE_CHANNEL, StringComparison.OrdinalIgnoreCase))
		await this.Phone.CallAsync(template, destination, data, cancellation:cancellation);
	else
		throw new ArgumentException($"Unsupported ‘{channel}’ channel.", nameof(channel));
}
```
{% endcode %}

## GUI Configuration Scenario

The following are interface design suggestions based on descriptor capabilities. Discussions has not yet implemented such a notification rule interface:

{% stepper %}
{% step %}
Read all `ITransmitter` in the service container and display the sender list, such as `Phone`, `Wechat`.
{% endstep %}

{% step %}
Read the `Descriptor.Channels` of the selected sender and display channels such as SMS, voice, and WeChat template messages.
{% endstep %}

{% step %}
Read `Templates` of the selected channel and display the template list and parameter definition.
{% endstep %}

{% step %}
Saves user-selected transmitters, channels, templates, destination expressions, and parameter mappings.
{% endstep %}

{% step %}
Construct `TransmitterHandler.Argument` at runtime and hand it to `TransmitterHandler` or call `ITransmitter.TransmitAsync` directly.
{% endstep %}
{% endstepper %}

This set of models is particularly suitable for scenarios where "templates are fixed, parameters change, and recipients are dynamic" such as verification codes, user binding mobile phones/emails, password retrieval, alarm notifications, and approval reminders.

## Related Resources

* [ITransmitter.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Communication/ITransmitter.cs)
* [ITransmitterArgumenter.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Communication/ITransmitterArgumenter.cs)
* [TransmitterDescriptor.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Communication/TransmitterDescriptor.cs)
* [TransmitterDescriptor.partials.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Communication/TransmitterDescriptor.partials.cs)
* [TransmitterHandler.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Communication/TransmitterHandler.cs)
* [TransmitterUtility.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Communication/TransmitterUtility.cs)
* [PhoneTransmitter.cs](https://github.com/Zongsoft/framework/blob/main/externals/aliyun/src/Telecom/PhoneTransmitter.cs)
* [Wechat Transmitter.cs](https://github.com/Zongsoft/framework/blob/main/externals/wechat/src/Transmitter.cs)
* [Secretor.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Security/Secretor.cs)
