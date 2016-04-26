# ActionML HarnessML

The ActionML HarnessML project is a scaffold for creating Machine Learning Applications. It is meant to have the infrastructure for arbitrary input in the lambda and kappa patterns, several data stores, and result servers. 

## Goals

 - Extreme flexibility to use ML code or define custom algorithms
 - Ease of deployment using a micro-service architecture with optional lambda or kappa streaming/online data flow
   - Containerized deployment of micro-services to match algorithm needs, like a recipe
   - A GUI to control, monitor and modify setup
   - A GUI for each algorithm's tuning, cross-validation results, algorithm specific analytics, A/B testing results
 - Made to produce nearly turnkey systems that are easy to deploy and manage
 - Made for programmer and data scientist customization
 - Built-in multi-tenancy to serve multiple models, algorithms, or datasets.
 - Auto-cluster setup
 - Supply several input mechanisms to pick from
 - Supply a Datastore and StreamingDatastore
 - Support parallel distributed model creation (as with Spark's MLlib)
 - Support efficient local model calculation thru streaming algorithms like Vowpal Wabbit or T-Digest, or single machine implementations like TensorFlow.
 - Implement several of the most broadly used algorithms like:
   - The Universal Recommender
   - The Correlated Cross-Occurrence (CCO) engine for correlation analysis, from Apache Mahout
   - The Page Variant Multi-armed Bandit for automated A/B Testing
   - Behavioral Search for augmenting search with personalized or group behavioral data 
   - Text Classification
   - ?

## Non-goals

 - A non-goal of this project is to expect users to customize code. Each algorithm will be self-contained and will make all available tuning and configuration parameters available for modification. We expect this to cover 90% of needed modifications, the other 10% may require code changes.
 - Another non-goal is to make the HarnessML plugin architecture simple and restrictive or to force a certain work-flow. The project is loose enough to accommodate most any work-flow and machine learning toolset.

## Setup Architecture

The HarnessML Architecture will be based on micro-services, wherever possible, and these will come with their own clusterable containerized configuration and setup. The current model for this is Docker containers and swarms with recipes that allow easy deployment of code to a wide range of hardware. 

The setup recipe will be algo specific but may share many of the same components between algorithms&mdash;for instance Kafka, Spark, and HDFS.

Think of deploying multiple algorithms as the union of micro-services needed for all. This flexibility comes from the container architecture and our decoupling of micro-services. Ease of deploying these combinations is a core goal of the project. 

## Input

Input is a Kafka stream of arbitrary JSON objects. The stream source is a data gathering application, which may be transforming logs, reading from files, or sending events from client code. The stream sync is an algorithm's data source, which will make all decisions about what to do with the data.

## Output

Output is the result of a REST query. The form of the query is algorithm specific and the result is also. The REST endpoint is dynamically attached to the HarnessML's result server and is also specific to the algorithm. The only enforced standard is that the result server must provide a REST resource id for addressing requests to the correct result service.

Optionally some algorithms may output files. 