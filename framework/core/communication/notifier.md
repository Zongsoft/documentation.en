---
description: "INotifier notifies the initiator of interface and implementation status."
icon: bell
---

# Notifier

`INotifier` is the notification trigger interface, used to trigger a named notification to a specified recipient. It is more general than `ITransmitter`: notification content, recipients, setting objects and return results are all defined by the specific implementation.

## Interface Member

| member | Description |
| --- | --- |
| `Notify` | Fire notifications synchronously. |
| `NotifyAsync` | Fire notifications asynchronously. |

Source: [framework/Zongsoft.Core/src/Communication/INotifier.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Communication/INotifier.cs#L38) (excerpt; see source for context).

{% code title="INotifier.cs" %}
```csharp
public interface INotifier
{
	/// <summary>激发一个通知给指定的接受者。</summary>
	/// <param name="name">指定要激发的通知名。</param>
	/// <param name="content">指定的通知内容。</param>
	/// <param name="destination">指定通知的接受者，由具体实现者定义支持的接受者类型。</param>
	/// <param name="settings">指定的通知设置。</param>
	/// <returns>返回通知结果对象，由具体实现者定义。</returns>
	object Notify(string name, object content, object destination, object settings = null);

	/// <summary>异步激发一个通知给指定的接受者。</summary>
	/// <param name="name">指定要激发的通知名。</param>
	/// <param name="content">指定的通知内容。</param>
	/// <param name="destination">指定通知的接受者，由具体实现者定义支持的接受者类型。</param>
	/// <param name="settings">指定的通知设置。</param>
	/// <returns>返回通知结果对象，由具体实现者定义。</returns>
	ValueTask<object> NotifyAsync(string name, object content, object destination, object settings = null);
}
```
{% endcode %}

## Differences from Transmitter

Both `INotifier` and `ITransmitter` can be used for "notifications", but the focus is different:

| abstract | focus | Suitable for the scene |
| --- | --- | --- |
| `INotifier` | Fires a named notification with goals, settings, and results defined by the implementation. | In-site notifications, session notifications, desktop push, and customized business reminders. |
| `ITransmitter` | Send templated messages by transmitter, channel, template and parameter. | Template SMS, voice notification, WeChat template message, and verification code sending. |

## Discussions Site Message Path

There is currently no complete implementation of INotifier available for reuse in Discussions and frameworks. Discussions uses MessageService to save the relationship between messages and receivers in the site; it is not an implementation of INotifier and is not responsible for pushing messages to online sessions in real time.

Source: [src/Services/MessageService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/MessageService.cs#L49) (excerpt; see source for context).

{% code title="MessageService.cs" %}
```csharp
public int Send(Message message, IEnumerable<uint> users)
{
	if(message == null)
		throw new ArgumentNullException(nameof(message));

	if(users == null || !users.Any())
		return 0;

	using(var transaction = new Transaction())
	{
		//插入消息
		if(this.Insert(message) < 1)
			return 0;

		//插入用户消息
		var count = this.DataAccess.InsertMany(users.Select(uid => new UserMessage(uid, message.MessageId)));

		//提交事务
		transaction.Commit();

		return count;
	}
}
```
{% endcode %}

Send receives the message object and user number collection, and writes the message and UserMessage in the transaction. The read status is updated by the MessageService's query hook. If you need to understand the actual sending process of universal template SMS, please continue reading [Transmitter](transmitter.md).

## Related Resources

* [INotifier.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Communication/INotifier.cs)
* [Communication source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Communication)
