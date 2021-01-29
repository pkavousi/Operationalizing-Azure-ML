# Operationalizing Machine Learning

## Table of contents

* [Overview](#Overview)
* [Architectural Diagram](#Architectural-Diagram)
* [Key Steps](#Key-Steps)
* [Screenshots](#Screenshots)
* [Screen Recording](#Screen-Recording)
* [Comments and future improvements](#Comments-and-future-improvements)
* [Dataset Citation](#Dataset-Citation)
* [References](#References)

***

## Overview

This project involves two parts:

* A machine learning production model is generated by AutoML in Azure Machine Learning Studio, and then its best model is deployed using Azure Container Instance (ACI). Swagger UI is used to import swagger.json file of the deployed endpoint and Apache Bench (ab) is used to benchmark it. Finaly, successful API requests are made to the endpoint with a jason payload.  
* The second part of the project repeats the same process by Azure Python SDK to create, train, and publish a pipeline using Jupyter Notebook in Azure ML Studio.

The used dataset is available from [here](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv) and contains marketing data about individuals. The data is for a marketing campaign (phone calls) with a _yes/no_ output as whether the client will subscribe a bank term deposit. Thus, the machine learning problem for this project is a binary classification.

***

## Architectural Diagram

The architectural diagram represents an overview of the operations. The diagram below is a visualization of the flow of operations from start to finish:

![Architectural Diagram](images/Architechture.jpg?raw=true "Architectural Diagram")  
  
***

## Key Steps

The project key steps are as following:

**Authentication:**
A security principal is required to be generated. However, since the Azure workspace is provisioned by Udacity, this action is not allowed. Thus, I skipped this step.

**Automated ML Experiment:**
A comupte cluster is generated to be used in AutoML along with the provided dataset.  

**Deploy the Best Model:**
The best model of the AutoML run is deployed using Azure Container Instance (ACI). The _Best Model_ will appear in the _Details_ tab, while it will appear first in the _Models_ tab. An HTTP weblink is generated for its endpoint.  

**Enable Logging:**
We choose the best model for deployment and enable "Authentication" while deploying the model using Azure Container Instance (ACI). The executed code in logs.py enables Application Insights. "Application Insights enabled" is disabled before executing ```logs.py```. Application Insight can also be activated when deploying the model using Azure ML Studio console.

**Swagger Documentation:**
Azure automatically generates a Swagger JSON file for the deployed model. Swagger UI is used to consume the endpoint and provide us with some insights into model input structure and its health check.

**Consume Model Endpoints:**
I used the ```endpoint.py``` script to interact with the deployed model. I run the script with the updated _scoring_uri_ that was generated after deployment and also the _key_ of the service. This URI is found in the_Details_ tab, above the Swagger URI and _key_ in under _Consume_ tab.

Apache Benchmark(ab) is used for load testing and benchmarking tool for Hypertext Transfer Protocol (HTTP) endpoint server.

**Create and Publish a Pipeline:**
The Jupyter Notebook is run with the same keys, URI, dataset, cluster, and model names as what has been created.

**Documentation:**
The documentation includes: 1. the [screencast](https://youtu.be/hswOvC9mt0I) that shows the entire process of the working ML application; and 2. this README file that describes the project and documents the main steps.

***

## Screenshots

### **Step 2: Automated ML Experiment**

The prerequisite for AutoML is that the dataset is uploaded in Datastore of the AzureML. This step is checcked as below:

**Registered Datasets:**

![Registered Datasets](images/Dataset.PNG?raw=true "Registered Datasets")

**Creating a new Automated ML run:**

I select the Bank-marketing dataset and in the second screen, I make the following selections:

* Task: _Classification_
* Primary metric: _Accuracy_
* _Explain best model_
* _Exit criterion_: 1 hour in _Job training time (hours)_
* _Max concurrent iterations_: 5. Please note that the number of concurrent operations **MUST** always be less than the maximum number of nodes configured in the cluster.

![Creating a new Automated ML run](images/BuildingAutoML.PNG?raw=true "Creating a new Automated ML run")

**Experiment is completed**

The experiment runs for about 30 min. Once completed, The resulting models are as following:

![AutoML completed](images/AML_Completed.JPG?raw=true "AutoML is completed")

**Best model**

In the _Models_ tab, the first model (at the top) is the best model.
The best model characteristics & metrics:

![Best model](images/Best_Model.JPG?raw=true "Best model")

![Best model metrics](images/Best_model_metrics.JPG?raw=true "Best model metrics")

The _Data guardrails_ tab, flags data for having an imbalanced output column:

![Imbalanced Data](images/Imbalanaced_data.JPG?raw=true "Imbalanced Data")

### **Step 3: Deploy the Best Model**

The best model is deployed with _Authentication_ enabled and using the _Azure Container Instance_ (ACI).
We can interact with the deployed model endpoint HTTP API service by GET and POST requests.
![Deployment](images/model_deploy.JPG?raw=true "Deployment")

### **Step 4: Enable Application Insights**

I enabled _Application Insights_ by adding the line `service.update(enable_app_insights=True)` to the `logs.py`:

**"Application Insights" enabled in the _logs.py_**
Application Insights is a feature of Azure Monitor and can be used to monitor your deployed endpoints. It will automatically detect performance anomalies, and help us to understand what users actually do with the deployed endpoint.  

!["Application Insights" enabled](images/model_deploy3.JPG?raw=true "'Application Insights' enabled")

Screenshot of the tab running "Application Insights":

!["Application Insights" graphs](images/Application_insight.JPG?raw=true "'Application Insights' graphs")

We can see Failed requests, Server response time, Server requests & Availability graphs in real time.

**Running logs.py script**

We can run the `logs.py` , where  _name_ is the name of the deployed model (_deployedmodel_) and I add the line `service.update(enable_app_insights=True)`:  

![Running logs.py script](images/deploy_log.JPG?raw=true "Running logs.py script")

### **Step 5: Swagger Documentation**

**Swagger** is a set of open-source tools built around the OpenAPI Specification that can help us design, build, document and consume REST APIs. One of the major tools of Swagger is **Swagger UI**, which is used to generate interactive API documentation that lets the users try out the API calls directly in the browser.

I use Swagger to consume the deployed endpoint. Azure provides a _Swagger JSON file_ for deployed models. This file can be found in the _Endpoints_ section, in the deployed model section. This file is downloaded and saved in the _Swagger_ folder.

The  `swagger.sh` and `serve.py` download and run the latest Swagger Docker container in (_swagger.sh_), and start a Python server on port 9000 (_serve.py_).

![Curl swagger.json](images/Swagger1.JPG?raw=true "Curl swagger.json")

![swagger.sh run](images/Swagger2.JPG?raw=true "swagger.sh run")

Replacing Explore field with : `http://localhost:9000/swagger.json`, Swagger UI generates interactive API documentation that provides the opportunity to make API calls directly in the browser.  

![Swagger runs on localhost](images/Swagger3.JPG?raw=true "Swagger runs on localhost")

We can see below the HTTP API methods and responses for the model:

**Swagger runs on localhost - POST/score endpoints**

![Swagger runs on localhost - POST/score endpoint](images/Swagger4.JPG?raw=true "Swagger runs on localhost - POST/score endpoint")

### **Step 6: Consume Model Endpoints**

I consume the deployed model endpoint using the `endpoint.py` script provided where I replace the values of `scoring_uri` and `key` to match the corresponding values that appear in the _Consume_ tab of the endpoint:  
**Consume Model Endpoints: running endpoint.py**

![endpoint.py](images/Consume1.JPG?raw=true "endpoint.py")

![run endpoint.py](images/Consume2.JPG?raw=true "run endpoint.py")

**Model Benchmarking: running benchmark.sh**

Apache Bench is use to benchmark the deployed endpoint. It provides load testing and benchmarking tool for Hypertext Transfer Protocol (HTTP) server. Apaache Bench needs the _key_ , _endpoit uri_, and ```data.json``` file, which is generated after running  `endpoint.py`.

![run benchmark.sh](images/Consume2.JPG?raw=true "Endpoint Benchmarking")

### **Step 7: Create, Publish, and Consume a Pipeline**

In this second part of the project, a Jupyter notebook is used with the same dataset, keys, URI, cluster, and model names which were created in the first part. `aml-pipelines-with-automated-machine-learning-step.ipynb`.  

The purpose of this step is to create, publish and consume a pipeline using the Azure Python SDK.  

**The Pipelines section of Azure ML Studio**  
The Pipeline is created using the notebook in this project. Subtasks are encapsulated as a series of steps within the pipeline. An Azure Machine Learning pipeline can be as simple as one that calls a Python script to an automatic Data Orchestration and Model Orchestration. However, in this project we just use Mode Orchestration.  

![Pipeline has been created](images/Pipeline1.JPG?raw=true "Pipeline has been created")

**Published Pipeline Overview showing a REST endpoint and an ACTIVE status**  
All published pipelines have a REST endpointwhich can be triggered a run of the pipeline from any external systems, including non-Python clients. This endpoint enables "managed repeatability" in batch scoring and retraining scenarios.

![Published Pipeline Overview showing a REST endpoint and an ACTIVE status](images/Pipeline6.JPG?raw=true "Published Pipeline Overview showing a REST endpoint and an ACTIVE status")
![ML studio showing the pipeline endpoint as Active](images/Active_pipeline.JPG?raw=true "ML studio showing the pipeline endpoint as Active")

**Jupyter Notebook: RunDetails Widget shows the step runs**   

RunDetails class can be used to show the running and non-running nodes of the pipeline.

![Jupyter Notebook: RunDetails Widget shows the step runs](images/Pipeline3.JPG?raw=true "Jupyter Notebook: RunDetails Widget shows the step runs")

![Jupyter Notebook: RunDetails Widget shows the step runs2](images/Pipeline2.JPG?raw=true "Jupyter Notebook: RunDetails Widget shows the step runs2")

**In ML Studio: Completed run**  
The following screenshots show the completed pipeline runs that are triggered using Jupyter notebook. It involves running AutoML and deploying its best model.

![In ML Studio](images/Pipeline5.JPG?raw=true "In ML Studio")

![In ML Studio](images/Pipeline4.JPG?raw=true "In ML Studio")

***
## Screen Recording

The screen recording can be found [here](https://youtu.be/hswOvC9mt0I) and it shows the project in action. More specifically, the screencast demonstrates:

* The working deployed ML model endpoint
* The deployed Pipeline
* Available AutoML Model
* Successful API requests to the endpoint with a JSON payload

***
## Comments and future improvements

* The data is **highly imbalanced** and an auxilary method such as Synthetic Minority Oversampling Technique, or SMOTE for short could make the prediction better.  

* Increasing the training time could help with model accuracy. This can increase costs and there must always be a balance between minimum required accuracy and assigned budget.  

* Lastly, a modularized version of the code can be made for CI/CD and deployments using Kubernetes

***
## Dataset Citation

[Moro et al., 2014] S. Moro, P. Cortez and P. Rita. A Data-Driven Approach to Predict the Success of Bank Telemarketing. Decision Support Systems, Elsevier, 62:22-31, June 2014.

***
## References

- Udacity Nanodegree material
- [App](https://app.diagrams.net/) used for the creation of the Architectural Diagram
- [SMOTE](https://arxiv.org/abs/1106.1813) Synthetic Minority Over-sampling Technique, 2002. 