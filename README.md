# HarnessML Proposal

The HarnessML project is a scaffold for creating Machine Learning Applications. 

It is meant to have the infrastructure for arbitrary input in the batch, lambda, and kappa patterns, several data stores, and result servers. These components are implemented as compossible microservices. Pick an algorithm and microservice options and you get a running production ready Machine Learning System complete with input server, ML magic, and result server. If you want file output, no problem, replace the ResultService microservice with a ResultWriter et voila. Since the microservices are compossible you can share them between algorithms&mdash;and some aren't so "micro", like say Spark.

As a shortcut you can think of HarnessML as the next generation of PredictionIO though it will depart from that project in several important ways.

## Goals

 - **Scaffolding**: Provide a set of generalized services and scaffolding to enable creation and deployment of production ready Machine Learning algorithms.
 - **Flexibility**: Extreme flexibility to use whatever ML code is convenient to define custom algorithms&mdash;not limited to Spark or any one technology
 - **Microservice Architecture**: Microservices and recipe based deployment
   - Completely based on either existing services or new microservices that are in every case containerized.
   - Microservices may have either a synchronous REST API or an asynchronous message based API&mdash;the later allowing for easy scalability.
   - Deployment of microservice through an external orchestration layer (possibly scripted Docker-machine, possibly Chef recipes, TBD)
   - Microservices will all have base discovery and auto connection build in to ease orchestration and allow services to be added to a running system.
   - Recipes for containerized deployment of microservices to match algorithm needs
   - Compossible recipes that share services and containers
   - Every plugin algorithm should include a recipe for automated deployment
   - **Optional**: A GUI to control, monitor and modify system setup
   - **Optional**: Hooks so an algorithm can provide its own plugin GUI for tuning, cross-validation results, and algorithm specific analytics or features
 - **Batch, Lambda, and Kappa** Support for batch, lambda (batch + realtime), or kappa (realtime streaming online) data flow
   - Support parallel distributed model creation (as with Spark's MLlib)
   - Support efficient local model calculation thru streaming algorithms like Vowpal Wabbit or T-Digest, or single machine implementations like TensorFlow.
 - **Turnkey Systems**: Made to produce nearly turnkey systems
   - Easy to deploy the system, because the components required for an algorithm are included in the algo recipe.
   - Easy to manage the system, because the overall system has a management GUI
   - Easy to deploy algorithms, each comes with a deployment recipe 
   - Easy to manage algorithms, because each algo may provide its own GUI
 - **Customization**: 
   - Made so an experienced programmer and/or data scientist can customize but beginner customization is not a goal. 
   - Good algorithms should be user configurable, through parameters, seldom requiring code customization 
 - **Debugger Friendly**: Made to be used with a debugger
 - **Flexible Input**: Several input mechanisms to pick from
   - Kafka stream input
   - File input
 - **Flexible Output**: Several output mechanisms supported
   - File output
   - REST output/result server
   - Each algo defines it's own input/output values within JSON bodies
 - **Multi-model, multi-tenant**: input/output
   - Multiple models addressed by ids (Topic ids or REST resource ids)
   - Auto-cluster setup, input and output servers will detect other instances and coordinate load sharing with the minimum of setup (similar to Elasticsearch)
   - TLS (SSL/HTTPS) and authentication supported optionally for all external APIs
 - **I/O Endpoints**
   - Auto load balanced
   - No single point of failure
   - TLS and auth supported
   - From the outside a running system looks a bit like an Elasticsearch cluster in that it operates like a peer based cluster with all nodes able to behave like a master to spread load and if any node goes down no data is lost.
   - Any node is able to handle requests to any algorithm or more precisely any dataset (data + model + algorithm) 
 - **First Algorithms**: Implement several of the most broadly used PredictionIO algorithms and ones that stretch the architecture to the limits of the batch - lambda - kappa spectrum using several ML libs and techniques
   - The Universal Recommender, based on Correlated Cross-Occurrence (CCO) engine from Apache Mahout&mdash;Lambda
   - Text Classification&mdash;Lambda
   - Bayesian Bandit for automated A/B Testing&mdash;Kappa
   - Behavioral Search for augmenting search with personalized or group behavioral data&mdash;Batch
   - ?

## Non-goals

 - Users are not expected to customize code. Each algorithm will be self-contained and will make all available tuning and configuration parameters available for modification. We expect this to cover 90% of needed modifications, the other 10% may require code changes but making this simple will not be a goal if flexibility has to be sacrificed.
 - Another non-goal is to make the HarnessML plugin architecture simple and restrictive or to force a certain work-flow. The project is loose enough to accommodate most any work-flow and machine learning toolset.

## Setup

The HarnessML Architecture will be based on microservices, wherever possible, and these will come with their own clusterable containerized configuration and setup. A possible model for this is Docker containers and swarms with recipes that allow easy deployment of code to a wide range of hardware and also provides for composing services where they are shared between algorithms.

The setup recipe will be algo specific but may share many of the same components with other algorithms&mdash;for instance Kafka, Spark, and HDFS may be required by more than one algorithm.

Think of deploying multiple algorithms as the union or composition of microservices needed for all. This flexibility comes from the decoupling of microservices. Ease of deploying these compositions is a core goal of the project. 

## Input

Two forms are supported, in each case the objects are encoded in JSON:

 - **Kafka Stream**: Input is a Kafka stream of arbitrary JSON objects. The stream source is a data gathering application, which may be transforming logs, reading from files, or sending events from client code. The stream sync is an algorithm's data source, which will make all decisions about what to do with the data.
 - **Files**: Batch or bootstrap input is supported. This is useful for restoring a backup where Kafka is not used for primary event record keeping.
 
## Lambda **and** Kappa

Without getting into semantics we mean algorithms with batch/offline model computation and pipelines that compute models online in realtime. To support both flavors we require that input streams have a mechanism for watermarking/checkpointing so that the state since the beginning of time can be encapsulated in a snapshot, from which replaying subsequent events will bring you to the same state as if all events had been played from the beginning of time. A round about way of saying that we require a way to snapshot the entire state of a running algorithm.

For a recommender this may involve a timeWindow of events and state of properties. For a kappa style multi-armed-bandit it might be only the MAB state/model. 

## Output

Output may be desirable in 3 forms and is, by default, JSON objects:

- **Result Server**: This output is on-demand and is the result of a REST query. The form of the query is algorithm specific and the result is also. The REST endpoint is dynamically attached to the HarnessML's result server and is also specific to the algorithm. The only enforced standard is that the result server must provide a REST resource id for addressing requests to the correct result service.
- **Kafka Stream**: This is most obvious when using streaming online algorithms that make notifications. 
- **Files**: in anything addressable as an HDFS file, so localFS, HDFS, S3, etc. This is useful as backup for possible restore. 

## HarnessML Microservices

The HarnessML scaffolding is made up of microservices that are useful for algorithm implementation but are not individually required. They may or may not be visible externally as APIs since they are meant for internal use. 

 - **Input server**: attaches to a Kafka topic stream or can be pointed to files.
 - **DataStore**: Interface to highly scalable NoSQL DB.
 - Output Server for query type output
 - **Management GUI**: Displays system information and provides hooks for algorithms to attach their own extension GUI.
 - **Kafka Source**: Parallel and distributed Kafka source writer (for push output like online streaming results)
 - **File Writer** Parallel and distributed file writer
 - **Data flow management?** Is this required? I doubt it but if so maybe something like Apache Beam? 

## Component Services

Though not strictly speaking "micro-services" several components will be supported for use by algorithms.

 - **Spark**
 - **HDFS**
 - **Database**: An easy to maintain highly scalable NoSQL DB to take the place of the current PredictionIO's HBase (opt. JDBC?). This may be Cassandra, Redis, JDBC (MySQL, Postgres) or other. It will be possible to replace the default with another.

## Algorithms and Recipes

Algorithm implementations take input and generates output. An algorithm contains or references all code needed to attach to input, and create output. The actual workflow depends on the algorithm as will the input/output form. So let's take 2 examples for illustration.

### The Universal Recommender

The UR is an example of a lambda style data flow. Generally this should be very similar to any lambda style algorithm.

The UR ingests indicators of user preference, and item metadata. This input can be seen as an immutable event stream. For practical reasons some events are reflected immediately into the model and others will only affect the model when a training operation is done. For other practical reasons we must implement a moving event window by either watermarking the stream periodically or truncating. The current implementation in PredictionIO watermarks item properties in the EventStore and truncates the usage event stream using the "SelfCleaningDataSource". Item properties are not reflected in realtime to the model but only updated on a training operation. The UR outputs results on-demand by being queried.

To implement in HarnessML the following data/work flow would be created in the lambda pattern. There is an ***input state***, which works in realtime, a ***train task***, which is scheduled or triggered on-demand, and an ***output state***, which is automatically updated after a train task.

In HarnessML these are implemented as follows:

 - ***input state***: The Input Server supplies a time series of events, some are indicator events, and some represent deltas to item properties. In the ***input state*** the algorithm stores the usage events as a time-window in the datastore and keeps the current state of the item or user properties directly in the model got realtime updates. The usage event can have TTL associated with them and are de-duped at training time. 
 
     The ***input state*** is the only state that needs to support watermarking. This is saving the base state to that a replay of the immutable event stream from the watermark onwards, will always produce that same overall state. The watermark is then defined as the aggregate state of usage and properties. Some usage event can have set/unset pairs just as property change events can. So the current collapsed state of there will define the watermark state.

 - ***training task***: The training task occurs on the time-window events saved in the datastore. It creates a new model, saves it to Elsaticsearch and swaps it in after indexing. The ***output state*** if active is not aware of the update. If the ***output state*** is not active it will be launched unless disabled.
 
 - ***output state***: When the output state is launched either from the ***training task*** or after a boot, it will attache the correct class to a running OutputServer or launch one itself. In either case the ***output state*** is active once the class for output is attached to the OutputServer. The UR may also attache a GUI serving class to extend the system GUI. All of this "attachment" is done through dynamic Spray REST routing. The GUI mechanism is yet to be defned

 - ***analysis task***: This task automates the above workflow to produce static parts of analytics data like MAP@k and usage event analysis. This task is optional and run rarely.