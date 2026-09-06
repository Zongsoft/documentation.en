---
description: "Combine Discussions' state actions with the framework's state machine to understand migration, handler, and completion phases."
icon: code-branch
---

# Zongsoft.Components.States

`Zongsoft.Components.States` provides lightweight state machine and state diagram models for expressing states, state vectors, state contexts, state handlers and state flows. It focuses on "whether an object can be migrated from the current state to the target state, and what processing needs to be triggered during the migration process." It is suitable for separate management of state judgment, side effects and persistence actions on business objects.

If you are not familiar with the design motivation of finite state machine (**FSM**), it is recommended to read these two articles first:

* [Out of control code and state machines (Part 1)](https://blog.zongsoft.com/dai-ma-shi-kong-yu-zhuang-tai-ji-shang): Explain why states, state drivers, transition decisions, and state diagrams can reduce the cognitive load of complex processes.
* [Out of control code and state machine (Part 2)](https://blog.zongsoft.com/dai-ma-shi-kong-yu-zhuang-tai-ji-xia): Using an expression resolver example to demonstrate how to derive a state diagram from syntax rules and implement it in code.

## Design Intent

Complex business processes tend to get out of control, usually not because each branch is difficult in itself, but because when status, conditions, side effects, and persistent updates are mixed together, local modifications will affect many hidden paths. The state machine model breaks the problem into several relatively independent parts:

* **Status** indicates which business stage the object is currently in, such as order pending, approved, or canceled.
* **state vector** represents a migration direction, that is, from the source state to the target state.
* **state diagram** describes which state vectors are allowed to occur and is responsible for reading and writing the current state of the object.
* **state context** saves the key, source status, target status, description and parameters of this migration.
* **status handler** carries side effects during the migration process, such as recording audit logs, sending notifications, and updating associated data.
* **State Machine** is responsible for creating the migration context, finding the handler and driving the state chart to perform the migration.

This layering allows state rules to be kept in the state diagram, side effects to be concentrated in the handler, and the business entry to only express "I want to migrate a certain object to a certain target state." It is not intended to replace all process codes, but to make the "map" of state migration clear first.

## Type Relationship

| Type | Description |
| --- | --- |
| `State<TKey, TValue>` | Represents the state value corresponding to an object key, holding the associated state diagram, object key and value. |
| `StateVector<T>` | Represents the migration vector from the source state to the target state. |
| `IStateMachine` / `StateMachine` | The state machine entry is responsible for running state migration and scheduling handlers. |
| `IStateDiagram<TKey, TValue>` / `StateDiagramBase<TKey, TValue>` | The state chart is responsible for determining whether the state can be migrated and providing the ability to read and write the current state. |
| `IStateContext<TKey, TValue>` / `StateContext<TKey, TValue>` | State migration context, holds object keys, migration vectors, descriptions and parameters. |
| `IStateHandler<TKey, TValue>` / `StateHandlerBase<TKey, TValue>` | The state migration handler is responsible for handling the side effects of migration; the default completion phase will call the context to write the target state. |
| `IStateHandlerProvider` | State handler provider, usually obtains handler collection from dependency injection container. |


## Discussions How to Handle Status Currently

Discussions defines ThreadStatus, but operations such as topic review, locking, and pinning are mainly expressed by independent Boolean fields and service methods. There is currently no StateDiagramBase derived class implemented, and the forum cannot be described as having integrated this set of state machines.

Source: [src/Services/ThreadService.cs](https://github.com/Zongsoft/Zongsoft.Discussions/blob/main/src/Services/ThreadService.cs#L103) (excerpt; see source for context).

{% code title="ThreadService.cs" %}
```csharp
public bool SetLocked(ulong threadId, bool value)
{
	return this.DataAccess.Update<Models.Thread>(new
	{
		IsLocked = value,
	}, Condition.Equal(nameof(Models.Thread.ThreadId), threadId) & GetIsModeratorCriteria()) > 0;
}
```
{% endcode %}

This action depends on the topic number and moderator qualifications. For such single-field operations, explicit service methods are intuitive; re-evaluate statechart abstraction when complex migration relationships between states, multiple collaborating handlers, and unified completion stages occur.

## How the Framework Actually Drives Migration

The following comes from the framework state machine itself and is an implementation reference, not a newly created business example:

Source: [framework/Zongsoft.Core/src/Components/States/StateMachine.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/States/StateMachine.cs#L87) (excerpt; see source for context).

{% code title="StateMachine.cs" %}
```csharp
public void Run<TKey, TValue>(State<TKey, TValue> state, string description, IEnumerable<KeyValuePair<object, object>> parameters = null) where TKey : struct, IEquatable<TKey> where TValue : struct
{
	ArgumentNullException.ThrowIfNull(state);
	var context = this.GetContext(state, description);

	if(context == null)
		return;

	if(parameters != null)
	{
		foreach(var parameter in parameters)
			context.Parameters.TryAdd(parameter.Key, parameter.Value);
	}

	var count = 0;
	var handlers = this.GetHandlers<TKey, TValue>();

	foreach(var handler in handlers)
	{
		if(count++ == 0)
			_stack.Push(context);

		state.Diagram.Transfer(context, handler);
	}
}
```
{% endcode %}

Run obtains the migration context, combines the parameters into the context, finds the handler, and then calls the state chart to execute the migration. The context is only pushed onto the stack when the first handler is found, so subsequent persistence cannot be assumed to be complete without a handler.

## Completion Stages and Transactions

Source: [framework/Zongsoft.Core/src/Components/States/StateMachine.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/States/StateMachine.cs#L115) (excerpt; see source for context).

{% code title="StateMachine.cs" %}
```csharp
private void Stop()
{
	if(_stack.Count == 0)
		return;

	var frames = _stack.Reverse().ToArray();

	using(var transaction = new Data.Transaction())
	{
		for(int i = 0; i < frames.Length; i++)
		{
			this.OnStop(frames[i]);
		}

		//提交事务
		transaction.Commit();
	}
}
```
{% endcode %}

The completion phase traverses the saved context, calls the handler in the ambient transaction to complete the logic and submits. The state chart is responsible for expressing allowed migrations and status reading and writing, and the handler is responsible for the business actions in migration. The caller must manage the state machine lifecycle and cannot just call Run and discard the instance.

## Questions to Answer Before Choosing

- Are states mutually exclusive enumerations, or are they independent properties? Locking and pinning can be established at the same time, but may not be suitable for forcing the same item.
- Which migrations are allowed to occur, who has the authority to initiate them, and how to check for concurrent database changes?
- Does the side effect occur in the migration phase or the completion phase, and how to roll back or compensate in case of failure?
- Who controls the lifecycle of a state machine instance, and does it need to be persisted across requests?

This state machine is not a persistent workflow engine and does not automatically provide distributed transactions, approval history, or permission verification for all resources. See [Discussions Posting Transactions](../../data/transactions.md) for business matters.

## Reference Implementation

[State machine directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Components/States) contains states, vectors, contexts, state diagrams, and handler contracts. This article no longer retains asset warehousing, asset status diagrams and asset services that do not exist in the repository.
