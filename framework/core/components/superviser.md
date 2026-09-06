---
description: "Monitor the lifecycle using the framework's SuperviserTest real-world objects and concurrent testing instructions."
icon: eye
---

# Superviser

Superviser manages monitored objects, searches and cancels monitoring by pressing keys, and notifies status changes through events. Discussions There is currently no monitor business process, so this page uses an existing use case for the framework SuperviserTest.

## The Meaning of Registration and Keys

Source: [framework/Zongsoft.Core/test/Components/SuperviserTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Components/SuperviserTest.cs#L17) (excerpt; see source for context).

{% code title="SuperviserTest.cs" %}
```csharp
private void Initialize(int count = 2)
{
	for(int i = 0; i < count; i++)
	{
		var name = $"S{i + 1}";
		_superviser.Supervise(name, new MySupervisable(name));
		_superviser.Supervise(name, new MySupervisable(name));
	}
}
```
{% endcode %}

The test class holds Superviser<string>, MySupervisable is the monitored object in the same test file. The same key is registered repeatedly here to verify that the monitor does not count repeatedly by the number of calls. The test object is not the device protocol client, and network reconnection behavior cannot be inferred based on this.

## Cancel Monitoring and Object Notification

Source: [framework/Zongsoft.Core/test/Components/SuperviserTest.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/test/Components/SuperviserTest.cs#L43) (excerpt; see source for context).

{% code title="SuperviserTest.cs" %}
```csharp
public void TestUnsupervise()
{
	this.Initialize(2);

	Assert.True(_superviser.Unsupervise("S1", out var observable));
	Assert.NotNull(observable);
	Assert.IsType<MySupervisable>(observable);
	Assert.Equal("S1", ((MySupervisable)observable).Name);
	Assert.True(((MySupervisable)observable).IsUnsupervised(TimeSpan.FromSeconds(10)));
	Assert.False(_superviser.Contains("S1"));

	Assert.True(_superviser.Unsupervise("S2", out observable));
	Assert.NotNull(observable);
	Assert.IsType<MySupervisable>(observable);
	Assert.Equal("S2", ((MySupervisable)observable).Name);
	Assert.True(((MySupervisable)observable).IsUnsupervised(TimeSpan.FromSeconds(10)));
	Assert.False(_superviser.Contains("S2"));

	Assert.Equal(0, _superviser.Count);
}
```
{% endcode %}

Canceling the operation returns the associated object, after which the key no longer exists. The test waits for the object to receive the cancellation monitoring callback, indicating that the callback and call return times need to be considered separately.

## Concurrent Behavior

The same framework test file also verifies concurrent registration of the same key, concurrent cancellation, and event count. Actual business should specifically check the ownership when repeated registration, cancellation and re-registration are interleaved, and Contains cannot be verified only in a single-threaded environment.

## Lifecycle and Applicable Boundaries

Monitoring lifecycle and failure thresholds describe object monitoring strategies and do not automatically equal network connection survival, process health, or business request success. When implementing monitored objects, it should be clear when data, errors and completions are reported, and how resources are released after monitoring is cancelled.

Monitors themselves should be managed by an explicit service lifecycle; do not create shared monitors for each query, and do not continue to treat the old object as the current connection after cancellation. See [Service ownership](../services/locating.md) and [Notifications](../common/notification.md).
