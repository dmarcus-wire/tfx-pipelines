# TFX Standard Components

TFX standard components come pre-packaged with TensorFlow, and are designed to help improve your pipeline development velocity. 

Each standard component is designed around common machine learning tasks, and encodes Google's ML best practices around tasks such as data monitoring and validation, training and serving data transformations, model evaluation, and serving.

# Data Management

## ExampleGen
[examplegen](./tfx-examplegen.png)
- is the entry point to your pipeline, that ingests data
- supports out-of-the-box ingestion of external data sources such as CSV, TF Records, Avro, and Parquet
- supports external ingestion of CSV, Avro, Parquet, and TF Record data sources
- This can be done across charted file systems using glob file patterns
- leverages Apache Beam for scalable, fault-tolerant data ingestion
- customizable to new input data formats and ingestion methods
- enables significant time savings for you and your Machine Learning Team during pipeline development, by having version data ingestion code, that you can reuse across different machine learning use cases
- advanced data management capabilities such as data partitioning, versioning, and custom splitting on features, or time
- As outputs, ExampleGen produces TF examples, or TF sequence examples which are highly efficient in performant data set representations, that can be read consistently by downstream components
- This comes with directory of management and logging by default to ensure ML best practices on consistent projects setup
- an output configuration is defined to split the raw data into Train and Dev splits, with an 80-20 split
- organizes data using several abstractions, with a hierarchy of concepts named spans, versions and splits
  - A span is a grouping of training examples. 
    - If your data is persisted on a file system, each span may be stored in a separate directory
  - A span may correspond to a day of data or any other grouping that is meaningful to your task
  - In the example on the image below, you can see that you're using an input configuration to either ingest pre-split data or split the data for you based on the glob file patterns
  - splitting method can also be specified to use features like time, or an ID field that indicates entities, like a customer ID
  - further customize which spans your pipeline operates over, as part of your experiments using the Range Config
    - For example, this is helpful during experimentation to benchmark different pipelines on fixed data split windows, and restrict your pipeline to train continuously over rolling time windows
  - Each span can hold multiple versions of data
    - For example, if you were to remove some examples from a span to clean up poor data quality, this could result in a new version of that span
  - By default, TFX components operate on the latest version within a span. Each version within a span can further be subdivided into multiple splits. 
    - The most common use case for splitting a span is to split it into Train, Dev, Test data partitions
- brings configurable, and reproducible data partitioning and shuffling into TF Records, a common data representation used by all components in your pipeline
- Just like code, the ability to version your data kit, is critical to effectively monitoring your data and model performance in production.

[examplegen-span](./tfx-examplegen-span.png)

## StatisticsGen

[statisticsgen](./tfx-statisticsgen.png)
- INPUT: take in TF examples from the ExampleGen component
- OUTPUT: produces a data set statistics artifact, that contains features statistics used for downstream components
- performs a complete pass over your data, using Apache Beam, and the TensorFlow Data Validation Library, to calculate summary statistics for each of your features over your configured Train, Dev and Test splits
- includes statistics like mean, standard deviation, quantile ranges, and the prevalence of null values
- benefits in pipeline
  - uses the TensorFlow Data Validation library that comes with pre-built utilities and best practices for calculating future statistics, and identifying data anomalies that can negatively affect your model's performance
  - uses Apache Beam to compute full-pass data set features statistics for downstream visualization and transformations
  - allows your pipeline to scale data set statistical summaries as your data grows, with built-in logging, and fault tolerance for debugging

## SchemaGen

[schemagen](tfx-schemagen.png)
- a description of your input data called a schema that can be automatically generated
- INPUT: reads the StatisticsGen artifact to infer characteristics of your input data from the observed feature data distributions
- OUTPUT: produces a schema artifact, a schema.proto description of your data's characteristics - the schema is an instance of schema.proto
- Schemas are a type of protocol buffer, more generally known as a protobuf
- benefits in pipeline
  - can be configured to automatically generate a schema by inferring types, categories in accepted ranges from their training data
  - can specify datatypes for feature values, whether a feature has to be present in all examples, what are the allowed value ranges for each feature, and other properties
  - an optional component that does not need to be run on every pipeline run, and instead it is typically run during prototyping and the initial pipeline run
  - key benefit of a schema file is that you're able to reflect your expectations of your data in a common data description for use by all of your pipeline components
  - enables continuous monitoring of data quality during continuous training of your pipeline

## ExampleValidator

[examplevalidator](tfx-examplevalidator.png)
- identifies any anomalies in the example data by comparing data statistics computed by the StatisticsGen pipeline component against a schema
- underpins Transform
- benefits in pipeline
  - it can detect different classes of anomalies in the data
  - it can perform validity checks by comparing data sets statistics against a schema, di-codifies expectations of the user
  - detect feature, train serving skew by comparing training and serving data
  - detect data drift by looking at a series of feature data across different data splits
  - When applying machine learning to real-world data sets, a lot of effort is required to preprocess your data into a suitable format
    - converting between formats
    - tokenizing and stemming texts
    - forming vocabularies
    - performing a variety of numerical operations such as normalization

## Transform

[transform](./tfx-transform.png)
[transform-details](./tfx-transform-diagram.png)

- performs feature engineering on the TF examples data artifact emitted from the ExampleGen component
- you can define feature transformations with TF Transform in TensorFlow operations
  - converting sparse categorical features into dense embedding features
  - automatically generating vocabularies
  - normalizing continuous numerical features
  - bucketizing continuous numerical features into discrete categorical features
  - enriching text features
- INPUTS:
  - the data schema artifact from the SchemaGen, or imported from external sources
  - TensorFlow transformations typically defined in a preprocessing function
- OUTPUT:
  - emits a saved model artifact, and it encapsulates feature engineering logic
- When executed, the saved model will Accepts:
  - TF Examples emitted from an ExampleGen component
- Emits:
  - the Transform feature data. 
  - a transform data artifact that is directly used by your model during training
- benefits in pipeline
  - brings consistent feature engineering at training and serving time to benefit your machine learning project
  - By including feature engineering directly into your model graph, you can reduce train-serving skew from differences in feature engineering, which is one of the largest sources of air and production machine learning systems
  - like many other TFX components, is also underpinned by Apache Beam, so you can scale up your feature transformations using distributed compute as your data grows

# Model Management

## Trainer

[tfx-trainer](./tfx-trainer-1.png)

[tfx-trainer](./tfx-trainer-2.png)

- trains a tensor flow model in a standardized model format
- supports TF1 estimators and native TF2 Keras models via the generic executor
- allows you to parameterize your training and evaluation arguments, such as the number of steps as shown in the example
- INPUTS:
  - a transform TF examples state artifact 
  - transform graph produced by the transform component
  - A data schema artifact from the SchemaGen component
    - that trainer checks its input data features against.
  - And your TensorFlow modeling code typically defined in a model.py file
- OUTPUTS:
  - produces at least one model for inference and serving in a TensorFlow saved model format
    - contains a complete TensorFlow program, including weights and computation
  - Optionally, another model for evaluation such as in a Val saved model we'll also be emitted. The Val saved model contains additional information that allows the TensorFlow model analysis library. To compute the same evaluation metrics defined in the model in a distributed manner over a large amount of data and user defined slices
- benefits in pipeline
  - brings standardization to your machine learning projects
  - format does not require the original model building code to run, which makes it useful for sharing and deploying
  - By exporting standardized model formats, you can more easily share your models on platforms like TF hub and deploy them across a variety of platforms. Such as the browser with TensorFlow JS, and mobile phones and edge devices, with TF light through the model rewriting library
  - inherit the benefits of using TensorFlow for accelerating your model training, such as the TF distribute APIs, for distributing training across multiple cores and machines
  - utilizing hardware accelerators like GPUs and TPUs for training

## Tuner

[tuner](./tfx-tuner.png)
- makes extensive use of the Python Keras tuner API for tuning hyperparameter
- modify the trainer configurations to directly ingest the best hyperparameters, found from the most recent tuner run 
- You typically don't run the tuner component on every run, due to the computational cost and time but instead configure it for one off execution.
- INPUTS:
  - transformed data
  - transformed graph artifacts
- OUTPUTS:
  - a hyper parameter artifact
- benefits in pipeline
  - tight integration with the trainer component to perform hyper parameter tuning in a continuous training pipeline.

## Evaluator

[evaluator](./tfx-evaluator.png)

- how do you know how well it performed?
- model performance evaluation as inputs
- will use the model created by the trainer and the original input data artifact
- INPUTS
  - use the model created by the trainer in the original input data artifact. 
    - It will perform a thorough analysis using the TensorFlow model analysis library. 
    - To compute mean machine learning metrics across data splits and slices, it's typically not enough to look at high level results across your entire data set. 
- OUTPUTS
  - two artifacts and evaluation metrics artifact that contains configurable model performance metrics slices
  - And a model blessing artifact that indicates whether the models performance was higher than the configured thresholds and that it is ready for production.
- benefits in pipeline
  - bringing standardization to your machine learning projects for easier sharing and reuse
    - many teams would write custom evaluation code that was buggy and hard to maintain with a lot of duplication and made it hard to replicate
  - the ability to get your pipeline from pushing the poor performing model to production and a continuous training scenario.
  - assured that your pipeline will only graduate a model to production when it has exceeded the performance of previous models

## InfraEvalutor

- used as an early warning layer before pushing a model to production
- focuses on the compatibility between the model server binary, such as TensorFlow serving, and the model ready to deploy
- it is the users responsibility to configure the environment correctly. And InfraValidator only interacts with the model server in the user configured environment to see whether it works well.
  - Configuring this environment correctly will ensure that InfraValidation, passing or failing indicates whether the model would be survivable in the production serving environment
  - 
- The name InfraValidator came from the fact that it is validating the model in the actual model serving infrastructure
- If evaluator guarantees that performance of the model, InfraValidator guarantees that the model is mechanically fine, and it prevents bad models from being pushed to production
- INPUTS
  - takes the saved model artifact from the trainer component
    - Launches a sandbox model server with the model and test whether it can be successfully loaded and optionally queried using the input data artifact from the example Gen component.
- OUTPUT
  - resulting output InfraValidation artifact will be generated in the blessed output in the same way that evaluator does.
- benefits in pipeline
  - InfraValidator brings an additional validation check to your T effects pipeline by ensuring that only top performing models are graduated production and that they do not have any failure causing mechanical issues.
  - brings standardization to this model infra check and is configurable to mirror model serving environments such as Kubernetes clusters and TF serving

## Pusher

[pusher](./tfx-pusher.png)

- used to push a validated model to a deployment target during model training or retraining.
- on one or more blessings from other validation componet as input to decide whether to push the model.
- These targets may be TensorFlow light if you're using a mobile application TensorFlow JS If you're deploying in a JavaScript environment. 
- Or TensorFlow serving, if you're deploying to Claudia platform where it Kubernetes cluster. 
- The pusher component brings the benefits of a production gatekeeper to your TFX pipeline. 
- INPUTS
  - Evaluator blesses the model if the new trained model is good enough to be pushed to production. 
  - InfraValidator blesses the model if the model is mechanically survivable in a production environment. 
- OUPUTS
  - As output a pusher component will wrap model versioning data with the train TensorFlow saved model for export to various deployment targets.
- benefits in pipeline
  - ensure that only the best performing models that are mechanically sound, make it to production.
  - Machine learning systems are usually a part of larger applications. So having a check before production keeps your applications more reliable and available.
  - pusher standardizes the code for pipeline model export for reuse and sharing across machine learning projects.

## BulkInferrer

[bulk-inferrer](./tfx-bulkinferrer.png)

- used to perform batch inference on unlabeled TF examples.
- typically deployed after an evaluator component to perform inference with a validated model, or after the trainer component to directly perform inference on an exported model. 
- currently performs in memory model inference and remote inference. 
- for your machine learning project to directly include inference in your pipeline.
  - Remote inference requires the model to be hosted on cloud AI platform.
- INPUTS
  - reads from the following artifacts, 
  - a trained TensorFlow saved model from the trainer component, 
  - optionally a model blessing artifact from the evaluator component.
  - Input data TF example artifacts from the example Gem component typically these would be unlabeled examples and the configured test data partition.
- OUTPUTS
  - generates a inference result proto, which contains the original features and the prediction results. 

## Pipeline nodes

[pipeline](./tfx-pipelinenode.png)

Pipeline nodes are special purpose classes for performing advanced metadata operations
such as 
1. importing external artifacts into ML metadata, 
2. performing queries of current ML metadata based on artifacts properties and their history.

Nodes


- ImporterNode
  - [importernde](./tfx-importernode.png)
    - The most common pipeline node is the importer node, which is a specialty effects node that registers an external resource into the ML metadata library, so downstream nodes can use the registered artifact as input. The primary use case for this node is to bring in external artifacts like a schema into the TFX pipeline for use by the transform in trainer components. 
      - The schema Gen component can generate a schema based on inferring properties about your data on your first pipeline run. However, you will adapt the schema to codify your expectations over time with additional constraints on the features. Instead of regenerating the schema for each pipeline run, you can use the importer node to bring a previously generated or updated schema into your pipeline.
- ResolverNode
  - [resolvernode](./tfx-resolvernode.png)
    - Resolver node is a special TFX node that handles special artifact resolution logistics that will be used as inputs for downstream nodes. 
    - The model resolver is only required if you are performing model validation in addition to evaluation. 
    - In the case above, we validate against the latest blessed model. 
    - If no model has been blessed before, the evaluator will make the current candidate model the first blessed model.
- [resolverlatest](./tfx-resolverlatest.png)
- LatestArtifactResolver
  - returns the latest and artifacts in a given channel
    - This is useful for comparing multiple run artifacts, such as those generated by the evaluation component.
- LatestBlessedModelResolver
  - returns the latest validated and blessed model. This is useful for retrieving the best-performing model for exporting outside of the pipeline for one-off evaluation or inference tasks outside of the TFX Pipelines goal, such as hosting a model for exporting outside of the pipeline