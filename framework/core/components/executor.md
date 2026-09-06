---
description: "Zongsoft.Components Executor executor, context, Feature characteristics and execution pipeline."
icon: bolt-lightning
---

# Execution Pipeline

The execution pipeline packages a piece of processing logic into a unified execution unit, and extends cross-cutting capabilities such as retry, fallback, circuit breaking, rate limiting, and timeout through context and Feature attributes. It is suitable for abstracting "calling a handler" into a composable, configurable, and testable runtime object.

`Executor` is responsible for packaging delegate, handler or context processing logic into `IExecutor`; `Feature` is responsible for declaring the strategy capabilities required for execution; `IFeaturePipelineBuilder` is responsible for building these features into actual pipelines. The business logic remains as an ordinary delegate or handler, and the strategy logic is uniformly borne by the pipeline.

This model is suitable for scenarios where cross-cutting strategies need to be placed at a unified entrance. The calling party is facing `IExecutor<TArgument>` or `IExecutor<TArgument, TResult>`, and does not need to know whether it is a delegate, a handler or an object loaded by a plugin.

## Key Types

| Type | Description |
| --- | --- |
| `IExecutor` | Executor interface defines context-based execution entry. |
| `IExecutor<TArgument>` | Actuator interface with parameter types. |
| `IExecutor<TArgument, TResult>` | Executor interface with parameter and return value types. |
| `IExecutorContext` | Execution context interface, carrying input, output, services and execution status. |
| `Executor` | The executor static factory provides `Build(...)` and global `Features` and `Pipelines` extension points. |
| `ExecutorContext<TArgument>`、`ExecutorContext<TArgument, TResult>` | Default execution context implementation. |
| `ExecutorBase<TArgument>`、`ExecutorBase<TArgument, TResult>` | Executor base class, supports filter and template methods. |
| `IFeature` | Marker interface for implementing features. |
| `IFeatureBuilder` | Feature builder interface, used to combine multiple execution strategies in a chain. |
| `IFeaturePipeline` | Execution feature pipeline interface. |
| `IFeaturePipelineBuilder` | Feature pipeline builder interface. |
| `FeatureBuilder` | The default feature is builder, which provides chained combination entry. |
| `Features.BreakerFeature` | circuit breaking feature. |
| `Features.FallbackFeature` | fallback feature. |
| `Features.RetryFeature` | Retry feature. |
| `Features.ThrottleFeature` | rate limiting feature. |
| `Features.TimeoutFeature` | Timeout feature. |

## Execution Model

`Executor.Build(...)` can wrap delegate, handler or context processing logic into `IExecutor`. If Feature is specified, the feature pipeline will be built through `Executor.Pipelines`, and the original execution logic will be wrapped in the pipeline for execution.

Source: [framework/externals/polly/samples/Program.cs](https://github.com/Zongsoft/framework/blob/main/externals/polly/samples/Program.cs#L207) (excerpt; see source for context).

{% code title="Program.cs" %}
```csharp
var executor = _features.Build<int>(OnExecuteAsync);
```
{% endcode %}

This is a Polly interaction example of the framework: commands such as retry and timeout modify _features, execute the command and then build an executor for processing integer input; OnExecuteAsync is defined in the same file. For a complete sample and its policy configuration, see [Task Scheduling and Resilient Execution](../../externals/execution.md). Discussions Currently there is no automatic retry configured in the outer layer of post writing, and the writing process including file and statistics updates cannot be directly replayed.

In this structure, the business code only expresses "what to execute". Strategies such as retry, fallback, circuit breaking, and rate limiting are provided by the Feature pipeline and do not need to be scattered in the business code.

## Working with Polly

The core framework only defines the Feature model and pipeline construction extension point, and does not directly bind a certain elastic processing library. `Zongsoft.Externals.Polly` will convert Features in the core framework into Polly policy pipelines, allowing applications to gain specific execution capabilities such as retry, timeout, and circuit breaking while retaining a unified abstraction.

{% hint style="info" %}
If the corresponding `IFeaturePipelineBuilder` implementation is not registered, Feature is just a set of policy descriptions and cannot automatically generate retry, rate limiting or circuit breaking behaviors. The actual effect depends on the extensions and configuration loaded by the host.
{% endhint %}

## Use Cases

* Add timeouts, retries and circuit breaking to the processing logic when calling external interfaces.
* Pack the message handler into a unified executor to facilitate unified recording of logs and exceptions.
* Reuse the same execution context in device collection, event processing, and scheduling tasks.
* Add consistent execution strategies for handlers loaded by plugins.
* Separate policy configuration and business logic so that the caller only cares about the execution entry.

If it is just a local method call and there is no reuse strategy or unified context requirements, it is often simpler to call the business method directly. The execution order and specific behavior of Features depend on the pipeline builder implementation. The impact of each strategy on the number of executions, return values, and exceptions should be observed in actual examples.

## Reference Implementation

* [Executor.cs](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Core/src/Components/Executor.cs)
* [Features source code directory](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core/src/Components/Features)
* [Zongsoft.Externals.Polly](https://github.com/Zongsoft/framework/tree/main/externals/polly)
