# TFX Standard Components

TFX standard components come pre-packaged with TensorFlow, and are designed to help improve your pipeline development velocity. 

Each standard component is designed around common machine learning tasks, and encodes Google's ML best practices around tasks such as data monitoring and validation, training and serving data transformations, model evaluation, and serving.

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
