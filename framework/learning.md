---
description: "Understand the machine learning module's datasets, estimator catalog, and training pipeline, as well as the current implementation scope."
icon: graduation-cap
---

# Machine Learning

`Zongsoft.Learning` Build datasets, loaders, estimator descriptions and pipeline configurations around ML .NET. It is suitable for studying how to organize training steps as plugin extensions, and currently cannot be directly regarded as a complete training platform.

## Differences from Large Language Model Calls

[AI Integration Module](intelligences.md) mainly connects to existing model services for inference. The machine learning module here focuses on how structured data is loaded, transformed, and how to describe training steps. A common training process includes preparing samples, splitting the training set and the test set, fitting the model, evaluating the error and saving the model; deploying the prediction service is a subsequent stage.

Don’t judge a model’s availability based solely on the results on the training set. Data leaks, changes in feature definitions, and inconsistencies in preprocessing during training/prediction will make offline indicators lose their reference value. The business should first determine the prediction goals and evaluation methods, and then select the algorithm.

## Main Components

| composition | function | Current considerations |
| --- | --- | --- |
| Dataset | Describe sources, fields, and load settings | Field metadata does not automatically equal text loader's column configuration |
| data loader | Convert external data to training data | There is a text file loading path, and the specific options are consumed by the loader. |
| Estimator description | Provide a name, parameters, and builder for the transformation or training step | Discover registered capabilities through catalog |
| pipeline | Describe multiple steps in sequence | There are limitations to the current build implementation, see below |

Capabilities such as text file loading, column merging, one-hot encoding and hash encoding, and LightGBM regression are visible in the current directory. The directory structure is more important than mere algorithmic enumeration: extenders register steps by describing objects and builders, and consumers refer to them by name.

## Verify the Plugin Directory First

Discussions does not have machine learning use cases, and Learning does not have independent complete training examples. You can first understand the directory registration by referring to the existing plugin list of the framework: Next, mount the LightGbmRegressionTrainer as the LightGbm trainer in the Regression category.

Source: [framework/Zongsoft.Learning/src/Zongsoft.Learning.plugin](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Learning/src/Zongsoft.Learning.plugin#L46) (excerpt; see source for context).

{% code title="Zongsoft.Learning.plugin" %}
```xml
<extension path="/Workbench/MachineLearning/Pipeline/Regression">
	<object name="LightGbm" type="trainer" builder="{static:Zongsoft.Learning.Trainers.LightGbmRegressionTrainer.Instance, Zongsoft.Learning}" />
</extension>
```
{% endcode %}

If the directory is missing, first check the plugin manifest, extension registration and assembly deployment, and then check the training parameters. Text loading should explicitly check delimiters, header rows, column indexes, field types, and missing value handling, and do not assume that the display fields of the dataset will automatically configure the loader.

## Current Implementation Limitations

Currently, the Populate override of TextFileLoader.Settings still uses System.Reflection.PropertyInfo, while the corresponding virtual method of [Core](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Core) already uses System.Reflection.MemberInfo; there is a signature incompatibility between the two source code version combinations, so you should first check the build results and reference versions. PipelineController also only declares Area and HttpGet, so it cannot be inferred that there are complete reachable routes just by relying on the default MapControllers.

{% hint style="warning" %}
🚨 The current source code's `Pipeline.Build` calls `estimator.Append(estimator)` in subsequent steps, and the results are not accumulated back into the pipeline; therefore, the multi-step configuration cannot be deemed to have been correctly concatenated based on this. Empty step lists return empty results, and unknown step names also lack complete error conversions. These paths need to be fixed and verified before formal training.
{% endhint %}

The controller skeleton in the web project also does not constitute a complete training task API. Deployment packages alone do not get training scheduling, task persistence, model version repository, evaluation dashboard, or online prediction service. Design files in the database directory cannot be used as completed running functions.

## How to Carry Out Application Integration

1. Choose a small, repeatable data set with clear input columns, target columns, and evaluation metrics.
2. Verify loader output and independently verify a transformation or training step.
3. Confirm the step sequence and combination results against the current pipeline implementation, and then combine the steps after fixing the limitations.
4. The application is responsible for data version, training log, model storage and prediction entry; training failure and business request failure are handled separately.

When you need a directly runnable ML .NET training path, you should also refer to the official examples of the ML .NET version used, and verify the model results in the application. The metadata declaration of this module cannot be regarded as evidence of successful training.

Source references: [Machine learning module](https://github.com/Zongsoft/framework/tree/main/Zongsoft.Learning), [Pipeline construction](https://github.com/Zongsoft/framework/blob/main/Zongsoft.Learning/src/Pipeline.cs).
