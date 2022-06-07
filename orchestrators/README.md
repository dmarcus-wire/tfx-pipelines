# TFX Orchestrators

TFX orchestrators are responsible for scheduling TFX pipeline components, sequentially based on a directed graph of artifact dependencies.

## Why orchestrate your ML workflows?

Orchestration is about bringing standardization and software engineering best practices to machine learning workflows, so you can spend more time focusing on solving your machine learning problem.

Standardizing your machine learning pipelines, project setups with versioning, logging and monitoring, enables code sharing and reuse. And allows your pipeline to portably run across multiple environments. Production machine learning ultimately it's a team sport. Standardization allows machine learning teams to more easily collaborate.

Standardization allows machine learning teams to more easily collaborate.

New team members familiar with TFX pipelines and components also have an easier time ramping up on projects and making effective contributions.

Orchestrating machine learning workflows in a standardized way, is a key technique that Google has applied to industrialized machine learning

### Without orchestration...

by not adopting standardized machine learning pipelines. Data Science and machine learning engineering teams will face unique project setups, arbitrary log file locations, unique debugging steps that quickly accumulate costly technical debt.

## Process
1. Notebooks
   1. experimentation typically begins in a Jupyter Notebook. You can build your pipeline iteratively using an interactive context object
   2. The interactive context object also sets up a symbol in memory, ML metadata store using SQL lite and automatically stores and organized pipeline artefacts on the local file system.
   3. you can also export your local TFX pipeline to a production ready orchestrators such as Apache Beam with minimal code change
2. Orchestrators
   1. Although notebooks are great for experimentation and interactive execution, they do not have a level of automation for continuous training and computation necessary for production machine learning.
   2. Different orchestrators are needed to serve as an obstruction that seats over the computing environment that supports your pipeline scaling with your data.
   3. apache airflow, kubeflow pipelines and apache beam
   4. term DAG Runner to refer to an implementation that supports an orchestrator.
   5. no matter which orchestrator you choose, TFX produces the same standardized pipeline directed a cyclical graph

### Apache Beam
- Apache Beam direct runner to orchestrate and execute the pipeline DAG
- can be used for local debugging without incurring the extra airflow or kubeflow dependencies which simplifies system configuration and pipeline debugging
- great option for extending TFX notebook based prototyping
- package your pipeline defined in a notebook into a pipeline.py file using the interactive context export to pipeline method. You can then execute that file locally using the direct runner to debug and validate your pipeline before scaling your pipelines, data processing an a production orchestrator

### KubeFlow Pipelines
- Kubeflow is an open source machine learning platform, dedicated to making deployments of machine learning workflows on Kubernetes, simple, portable and scalable
- Kubeflow pipelines services on Kubernetes include the hosted ML metadata store container based orchestration engine, notebook server. And UI to help users develop, run and manage complex machine learning pipelines at scale, including TFX.
- From the UI you can create or connect to an easily scalable Kubernetes cluster for your pipelines, compute and storage.

### Apache Airflow
- platform to programmatically author schedule and monitor your workflows.
- The airflow scheduler executes tasks on an array of workers while following the specified dependencies.
- Rich command line utilities may complex graph construction and update operations on TFX pipeline DAGs easily accessible.
- Airflow is the more mature orchestrator for TFX with the flexibility to run your pipeline, along with pre pipeline data processing pipelines, and post pipeline model deployment in model prediction logic. 
- airflow is a more general orchestrator for the entirety of your machine learning system

### TFX CLI
- TFX command line utility for manual pipeline orchestration tests is important to mention.
- performs a full range of pipeline actions using pipeline orchestrators such as airflow, beam, and kubeflow pipelines
- you can use the CLI to create, update, and delete your pipelines
- Run a pipeline and monitor the run on various orchestrators
- as well as copy over template pipelines to modify and launch in order to accelerate your pipeline development.

# Apache Beam

- learn Beam, write your code once, and don't worry about scaling your data processing again
- provides a unified framework for running batch and streaming data processing, jobs that run on a variety of execution engines.
- why learn it? it's incredible flexibility and unification of batch and stream data processing that underpins TFX pipelines
- built on beam
  - TensorFlow Data Validation is built on the Beam programming model for batch computation of whole data-set statistics. 
  - Transform, is also built on the Beam programming model for batch feature engineering and feature transformations.
  - TensorFlow Model Analysis is built on the Beam programming model for batch and streaming model evaluation metrics across data splits and slices
  - Beam also powers BulkInference via the BulkInferer component
- The beam software development kit can work with numerous runtimes, including Full Operating System, Python, or Docker Containers and translate that to distributed compute cluster via runner.
- Beam abstracts away an incredible amount of complexity to distribute your pipelines data processing, which enables you to focus on improving your model's performance and delivering business impact.
- provides a portable API to TFX for building sophisticated data-parallel processing pipelines across a variety of execution engines or runners.
- brings a unified framework for batch and streaming data that balances correctness, latency, and costs and large unbounded out of order, and globally distributed data-sets.
- Apache Beam contains several key primitives. 
  - A pipeline object and its accompanying set of pipeline execution options. 
  - A PCollection abstraction, which represents a potentially distributed multi-element data-set. 
    - You can think of a PCollection as pipeline data. 
    - Beam transforms use PCollection objects as inputs and outputs. 
  - PTransforms, change, filter, group, analyze, or otherwise process the elements in a collection. 
    - A transform creates a new output PCollection without modifying the input collection. 
    - A typical pipeline applies subsequent transforms to each new output PCollection in turn, until processing is complete. 
  - ParDO is the core parallel processing transform in the Apache Beam software development kit, invoking a user-specified function on each of the elements in an input PCollection independently and possibly in parallel. 
    - Think of PCollections as variables and PTransforms as functions apply to these variables. 
    - The shape of the pipeline can be arbitrarily complex processing graph IO transforms. 
    - The final transform PCollections to an external source. 
    - Beam provides read and write transforms for several common data storage types. 
  - Lastly, runners are the actual software that except pipelines and translate them into massively parallel big data processing systems. 
    - TFX runners inherit from a specialized TFX runner class that the runners for local testing and debugging, and those for large-scale systems such as the Kubeflow DAG runner
  - Let's step through each action. 
  - First, Beam pipeline is created in and IO transform in just raw data into a Pcolection. Second, Beam transform maps a user textfile decoding function to the text data Pcollection. Third, TensorFlow transform, analyze and transform data-set function applies a user-defined pre-processing function for feature engineering. Finally, the transform texts Pcollection is written out as TF records. Each beam supported component is translated into a Beam graph. 