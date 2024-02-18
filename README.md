# cubejs-helm-charts-kubernetes
This repo contains the HELM charts for deploying the [**CubeJS**](https://cube.dev/) in Kubernetes clusters.   
![Untitled (500 x 300 px) (1)](https://github.com/OpstimizeIcarus/cubejs-helm-charts-kubernetes/assets/144666848/a0cf05e3-4425-4745-a68a-5c60e824faf5)

## Abstract
Deplyogint the cubejs stack is nothing but deploying all of its components. 
there are totally 3 main components we would need to deploy as mentioned:  
* 1️⃣ **cube-api** (Deployment)
* 2️⃣ **cube-refresh-worker** (Deployment)
* 3️⃣ **cubestore** (StatefullSet) --> **cubestore-worker ➕ cubestore-router**

##  What are we gonna deploy ???

This HELM charts is deployed in `cubejs` namespace with its own `serviceaccount`

| cubejs components  | Deployment | Statefull Sets | Services(ClusterIP) | Configmap | PVC(volumeclaims) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `cube-api` | ✔ | ❌ | ✔ | ✔ | ✔ | 
| `cube-refresh-workes` | ✔ | ❌ | ✔ | ✔ | ✔ | 
| `cubestore-worker` | ❌ | ✔ | ✔(headless) | ✔  | ✔ | 
| `cubestore-router` | ❌ | ✔ | ✔  (headless) | ✔ | ✔ | 

>[!NOTE]
>There will be in total ONLY 2 PVC created . one will be used for both cube-api and cuberefresh-workers and another will be used for both cubestore-router and cubestore-worker.  
>Also note that PVC for cube-api and cube-refresh-worker can be optional but for cubestore-router and cubestore-worker pvc is required as they will store pre-aggregated data. otherwise these pre-aggregated data will be stored in docker volume and we might loose the data incase of pod restart.


## Docker Images

There are 2 docker images given below which we will be used in the helm charts to deploy cube-api,cube-refresh-worker,cubestore-worker and cubestore-router  
* 🐳**cubejs/cube** : cube-api and cube-refresh-worker
* 🐳**cubejs/cubestore** : cubestore-worker and cubestore-router  

Now you may be 🤔thinking that using only 2  docker image , how are we gonna 4 different components of cubjes right ❓❓❓❓❓  
This is achieved by setting required **ENVIRONMENT** variables so that we can deploy all of these 4 different components of cubejs with just 2 docker images.

## Environment variables

Cubejs provides lot of ENVIRONMENT variables and each one has it own purpose. Below are the minimal environment configurations we will be using to deploy all the components of cubejs in kubernenetes.
For more information about all the available environment variables , visit : https://cube.dev/docs/reference/configuration/environment-variables  

* **🐳cubejs/cube : cube-api  and cube-refresh-worker**
  
| Environment Variable | cube-api | cube-refresh-worker | Description |
| :--- | :--- | :--- | :--- | 
| `CUBEJS_DB_TYPE` | ✔ | ✔ | A database from the list of [supported databases](https://cube.dev/docs/product/configuration/data-sources) |
| `CUBEJS_REFRESH_WORKER` | ✔ | ❌ | If `true` docker container acts as cube refresh worker |
| `CUBEJS_DEV_MODE` | ✔ | ✔ | If `true`, enables [development mode](https://cube.dev/docs/product/configuration#development-mode)
| `CUBEJS_DB_HOST` | ✔ | ✔ | The host URL for a database |
| `CUBEJS_DB_NAME` | ✔ | ✔ | The name of the database to connect to |
| `CUBEJS_DB_USER` | ✔ | ✔ | The username used to connect to the database |
| `CUBEJS_DB_PASS` | ✔ | ✔ | The password used to connect to the database |
| `CUBEJS_DB_SSL` | ✔ | ✔ | If `true`, enables SSL encryption for database connections from Cube |
| `CUBEJS_DEFAULT_API_SCOPES` | ✔ | ✔ | [API scopes](https://cube.dev/docs/product/apis-integrations/rest-api#configuration-api-scopes) used to allow or disallow access to REST API endpoints |
| `CUBEJS_CUBESTORE_HOST` | ✔ | ✔ | The hostname of the Cube Store deployment |
| `CUBEJS_LOG_LEVEL` | ✔ | ✔ | The logging level for Cube - `trace`,`error`,`info`,`warn` |


* **🐳cubejs/cube : cubestore-router and cubestore-worker**
  
| Environment Variable | cubestore-router | cubestore-worker | Description |
| :--- | :--- | :--- | :--- | 
| `CUBESTORE_WORKERS` | ✔   | ✔   | A comma-separated list of address/port pairs for Cube Store workers |
| `CUBESTORE_SERVER_NAME` | ✔ | ✔ | The full name and port number of the Cube Store server |
| `CUBESTORE_REMOTE_DIR` | ✔ | ✔ |  path on the local filesystem to store metadata and datasets from all nodes as if it were remote storage. Not required if using GCS/S3. Not recommended for production usage |
| `CUBESTORE_WORKER_PORT` | ❌ | ✔ | The port for Cube Store workers to listen to connections on. When set, the node will start as a `cube store worker` in the cluster |
| `CUBESTORE_META_ADDR` | ❌ | ✔ | The address/port pair for the router node in the cluster |
| `CUBESTORE_META_PORT` | ✔ | ❌ | The port for the router node to listen for connections on. Ignored when `CUBESTORE_META_ADDR` is set |
| `CUBEJS_LOG_LEVEL` | ✔ | ✔ | The logging level for Cube - `trace`,`error`,`info`,`warn` |  



## 💡[Recommended]Creating custom docker image for cube-api and cube-refresh worker
When deploying the cube-api and cube-refresh-worker , we need to make sure that the **cube.js** and [data model folder](https://cube.dev/docs/product/data-modeling/syntax#folder-structure) are mouted at **`/cube/conf`** location.

As in development stage, these data model folder will be updated frequently, so we need to make sure the latest files are available at **`/cube/conf`** for cube api instance and cube refresh worker.  
It's better to create a docker file and build our own custom image while deploying the cube api instance and cube refresh worker to reduce the operational/development overhead. 

```docker
# Refer the cubejs/cube:latest as base image
FROM cubejs/cube:latest

# copy all the models files to /cube/conf/model
COPY <CUBEJS_DATA_MODEL_FILES_DIRECTORY> /cube/conf/model/

# copy all the cube.js files to /cube/conf/
COPY <cube.js_FILE_LOCATION>/ /cube/conf/

```
>[!NOTE]
>Alternativly we can **init-container** which can pull the data-model folder and mount it under **/cube/conf** as well

## Usage

🧁 Clone this Helm Repository  
🧁 Update the values.yaml file  
| Environment Variable | Description |
| :--- | :--- |
| `CUBEJS_DB_HOST` |  The host URL for a database |
| `CUBEJS_DB_NAME` |  The name of the database to connect to |
| `CUBEJS_DB_USER` |  The username used to connect to the database |
| `CUBEJS_DB_PASS` |  The password used to connect to the database |

>[!NOTE]
>If you are going to create your own docker image for cube-api and cube-refresh-worker as mentioned in the [Creating custom docker image for cube-api and cube-refresh worker](#creating-custom-docker-image-for-cube-api-and-cube-refresh-worker), then ensure to update the image value for cube-api and cube-refresh-worker in values.yaml file  
  
🧁 Deploying the helm chart
```ruby
helm install cubejs
```



