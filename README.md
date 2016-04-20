# ActionML Harness

The ActionML Harness project is a scaffold for creating Machine Learning Applications. It is meant to have the infrastructure for arbitrary input in the lambda and kappa patterns, several data stores, and result servers. The goals of the project are:

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
   - The Correlated Cross-Occurrence (CCO) engine for correlation analysis
   - The Page Variant Multi-armed Bandit for automated A/B Testing)
   - Behavioral Search for augmenting search with personalized or group behavioral data 
   - Text Classification
   - ?
 
A non-goal of this project is to expect users to customize code. Each algorithm will be self-contained and will make all available tuning and configuration parameters available for modification. We expect this to cover 90% of needed modifications, the other 10% may require code changes.

Another non-goal is to make the Harness plugin architecture simple and restrictive or to force a certain work-flow. The project is loose enough to accommodate most any work-flow and machine learning toolset.