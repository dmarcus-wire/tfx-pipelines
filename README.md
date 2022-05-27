# tfx-pipelines
TFX Pipelines learning based on ML Pipelines on Google Cloud

Author: Ajay Hemnani

"ML is still an emerging engineering discipline and is a highly active area of research and tooling development."

"TFX represents the culmination of learned best practices around data and model management from over a decade of industrial-scale machine learning serving billions of Alphabet users. By implementing your machine learning workflows with TFX, you are leveraging the machine learning best practices and tooling used at Google to improve your project's probability of success. "

# Overview
1. Intro to TFX
2. Pipelines components and orchestrations. 
   1. TFX orchestrators bind ML pipelines together, and run components in a given sequence to achieve specific tasks. Such as data preparation model training and model evaluation.
   2. custom components allow for flexibility of ml pipelines
   3. automate your pipeline through continuous integration and continuous deployment
3. ML Metadata
   1. how to manage ML metadata, which is a critical piece of the process to allow for consistency. And repeatability during ML experimentation and development.
4. Reuse Pipelines
   1. reuse ML pipelines across multiple ML frameworks
   2. single KubeFlow pipelin for containerized models built in the most popular frameworks such as TensorFlow, Pytorch socket learn and XG boost. 
   3. ou will also learn how to train such models in parallel in the Kubeflow pipeline, and how to implement scheduled runs to perform continuous training of your models. 
5. Cloud Composer to orchestrate your continuous training pipelines. 
6. Use ML flow for managing the complete machine learning life cycle. 
   1. ML flow is an open source project which is developing a large contributory community.

# TensorFlow Extended (TFX)
- TFX is a Google production-scale machine learning platform based on the TensorFlow ecosystem, widely used internally at Google and fully open-sourced in 2019.
- It provides a flexible configuration framework and shared libraries to integrate common machine-learning tests, implemented this components needed to define, launch, and monitor your machine learning system. 
- It is designed to orchestrate your machine learning workflow with portability to multiple environments and orchestration frameworks in mind. This includes Apache Airflow, Apache Beam, and Kubeflow.
- TFX already supports four standard deployment targets for TensorFlow models. 
  - Deployment to TF Serving a high-performance Production machine-learning model server for batch and streaming inference. 
  - Deployment through a TF light model converter for inference on IoT and mobile devices. 
  - Deployment to web browsers through TensorFlow JS for lowly and C web applications. 
  - Deployment to TFO, a model repository for model sharing and transfer learning.
- TFX enables you to run multiple pipelines and parallel locally or in the Cloud
- This includes pipelines with different sets of data splits, models, and hyperparameters to iterate towards an optimal model that you can deploy to production faster.
- Automate your machine learning operational processes for individual and multiple machine learning pipelines.
- TFX is the most widely used machine learning platform at Alphabet. It is powering tens of thousands of user and programmatic machine learning pipelines at Alphabet subsidiary companies like DeepMind, Verily, and Waymo.

## Customers
- TFX enables global machine learning use cases such as Twitter's Cortex machine learning platform, which incorporates TFX to support large-scale machine learning applications such as tweet ranking for its a 100 million-plus users. 
- Airbus is also using TFX pipelines to automate large-scale anomaly detection and report generation for human reviewers of thousands of International Space Station sensors aboard the Columbus scientific laboratory. 
- SAP concurs using TFX to simplify its Bert natural language model deployments for conversational agents.

# Components

- A TFX component is an implementation of the machine learning task in your pipeline. They are designed to be modular and extensible while incorporating Google's machine learning best practices on tasks such as data partitioning, validation, and transformation.
- Each step of your TFX pipeline, again called a component, produces and consumes structured data representations called artifacts. Subsequent components in your workflow may use these artifacts as inputs. 
- Components are composed of five elements. 
  - Component specification or components specs define how components communicate with each other. 
    - They describe three important details of each component: 
      - it's input artifacts
      - output artifacts
      - runtime parameters that are required during component execution
  - Driver class which coordinates compute job execution, such as reading artifact locations from the ML metadata store, retrieving artifacts from pipeline storage, and the primary executor job for transforming artifacts. 
  - Executor class, which implements the actual code to perform a step of your machine learning workflow, such as ingestion or transformation on dataset artifacts. 
  - Publisher, which packages the component specification and executer for use in a pipeline.
  - Interface packages the component specification, driver, executer, and publisher together as a component for use in a pipeline.

## Procedure
1. a driver reads the component specification for runtime parameters and retrieve the required artifacts from the ML metadata store for the component. 
2. an executor performs the actual computation on the retrieved input artifacts and generates the output artifacts. 
3. the publisher reads the components specification to log the pipeline component run and ML metadata and write the components output artifacts to the artifacts store. 

TFX pipelines are a sequence of components linked together by a directed acyclical graph of the relationships between artifact dependencies. They communicate through input and output channels. 

## Parameters

Parameters are inputs to pipelines that are known before your pipeline is executed. Parameters let you change the behavior of a pipeline or part of a pipeline through configuration protocol buffers instead of changing your components and pipeline code. 

For example, the number of workers is a runtime perimeter to your entire pipeline that you can fix. When prototyping your pipeline, you can run different pipelines in parallel with a fixed number of workers that have different components or models and benchmark their runtime, memory, and performance in order to inform further improvements in your pipeline.

## Metadata

TFX implements a metadata store using the ml metadata library, which is an open-source library to standardize the definition, storage, and querying metadata for ml pipelines.

For notebook prototypes, this can be a local SQL database, and for production Cloud deployments, this could be a managed MySQL or Postgres database.

ML metadata does not store the actual pipeline artifacts. TFX automatically organize and stores the artifacts on local file systems, on a remote Cloud storage file system for the consistent organization across your machine learning projects.

## Orchestrators

Orchestrators coordinate pipeline runs specifically component executers sequentially from a directed graph of artifact dependencies. Orchestrators ensure consistency with pipeline execution order, component logging, retries and failure recovery, and intelligent parallelization of component data processing. 

## Task-Aware

TFX pipelines are task aware, which means that they can be authored in a script or a notebook to run manually by the user as a task. A task can be an entire pipeline run or a partial pipeline run of an individual component and its downstream components. 

A key innovation of TFX pipelines is that they're both task and data-aware pipelines. Data-aware means TFX pipelines store all the artifacts from every component over many executions so they can schedule component runs based on whether artifacts have changed from previous runs. 

## 

TFX horizontal layers coordinate pipeline components. These are primarily shared libraries of utilities and protobufs for defining abstractions that simplify the development of TFX pipelines across different computing and orchestration environments. 

# Overview

1. At a high level, there are four horizontal layers to be aware of. An integrated frontend enables GUI-based controls over pipeline job management, monitoring, debugging, and visualization of pipeline data models and evaluations. 
2. Orchestrators come integrated with their own frontends for visualizing pipeline-directed graphs. Orchestrators run TFX pipelines with shared pipeline configuration code and produce. All orchestrators inherit from a TFX runner class. TFX orchestrators take the logical pipeline object, which can contain pipeline args components and a DAG, and are responsible for scheduling components of the TFX Pipelines sequentially based on the artifact dependencies defined by the DAG. 
3. There are also shared libraries and protobufs that create additional abstractions to control pipeline garbage collection, data representations, and data access controls. An example is the TFXIO library, which defines a common in-memory data representation shared by all TFX libraries and components in an I/O abstraction layer to produce such representations based on Apache Arrow.
4. Finally, you have pipeline storage. ML metadata records pipeline execution metadata and artifact path locations to share across components. You also have pipeline artifacts Storage, which automatically organizes artifacts on local or remote Cloud file systems.
