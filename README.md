# Let's build a web site in 10 minutes

## Prerequisite

- [Create a Google Cloud Project][link:create-gcp-project]
- Install [Google Cloud CLI][link:gcloud-cli]
- Install [Node.js][link:node_js]
    - recommend use [nvm][link:nvm] to manage your node environment.

[link:create-gcp-project]:https://cloud.google.com/resource-manager/docs/creating-managing-projects
[link:gcloud-cli]:https://cloud.google.com/pubsub/docs/quickstart-cli
[link:node_js]:https://nodejs.org/en/
[link:nvm]:https://github.com/creationix/nvm


## Ready, Set, GO!

Create a directory for our web site.

```sh
mkdir node-app
```

Init npm package with default settings
```sh
cd node-app
npm init -y # -y means say "yes" to all configuration questions.
```

Open `package.json` and add a start script

```diff
{
  "name": "node-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
+   "start": "node app.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

Install `Express`

```sh
npm install express
```

Create `app.js` file

```sh
touch app.js
```

Write some express hello world

```javascript
// app.js
const express = require('express')
const app = express()
const PORT = 3000

app.get('/', (req, res) => res.send('Hello World!'))

app.listen(PORT, () => console.log(`Example app listening on port ${PORT}!`))
```

Run it!
```sh
npm start
```


## Start A Cluster

__This step will cost you money!!__

```sh
gcloud container clusters create <cluster-name>
```
> this command might need a while.

You can check it out with GUI at : https://console.cloud.google.com/kubernetes


## Dockerize our app

Create Dockerfile
```sh
# in app dir
touch Dockerfile
```

```dockerfile
FROM node:10-alpine

WORKDIR /app

COPY package.json /app
RUN npm install

COPY . /app

EXPOSE 3000

CMD npm start
```

Build it and run the docker image.
```sh
# in node-app/
docker build --rm -t node-app .

# run the docker container
docker run --name node-app -p 3000:3000 node-app
```

## Push image to GCR (Google Container Registry)

> [Pushing image to GCR][link:gcr-usage]

Make sure the image is successfully built.
```sh
docker images
```

Tag the image.
```sh
# docker tag [SOURCE_IMAGE] [HOST_NAME]/[GCP_PROJECT_ID]/[IMAGE]:[TAG]
docker tag node-app gcr.io/<project-name>/node-app
```

Push the images to GCR
```sh
docker push gcr.io/<project-name>/node-app
```

Check the image is push.
```sh
gcloud container images list-tags gcr.io/<project-name>/node-app

# DIGEST        TAGS    TIMESTAMP
# 19c81f3c8114  latest  2018-10-01T14:12:44
```

[link:gcr-usage]:https://cloud.google.com/container-registry/docs/pushing-and-pulling?hl=en_US&_ga=2.46868133.-386356805.1537960264


## Config Kubernetes
At this time, our cluster should be built. (You can check it on [GCP K8s Engine][link:gcp-k8s-eng])
Create a file in `node-app/` named `deployment.yaml`

```sh
# in node-app/
touch deployment.yaml
```
This is the configuration file of k8s deployment. It basically tells K8s how to deploy this app.

```yaml
apiVersion: apps/v1 # k8s version
kind: Deployment    # configuration type
metadata:
  name: nodeapp     # name of our app
spec:
  replicas: 3       # how many container should be running at the same time. In GCP, the minimun is 3.
  selector:
    matchLabels:
      app: nodeapp  # Set the target app to run. (Find with a label named "app" with the value "nodeapp")
  template:
    metadata:
      labels:
        app: nodeapp # Template for expension
    spec:
      containers:
      - name: nodeapp
        image: gcr.io/node-deploy-test-214507/node-app:latest
        ports:
        - containerPort: 3000
```

Then, create another file named `service.yaml` which tells k8s how the app/service act.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeapp
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  selector:
    app: nodeapp
  type: LoadBalancerapiVersion: v1
kind: Service
metadata:
  name: nodeapp
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  selector:
    app: nodeapp
  type: LoadBalancer
```

# Deploy the service

Make sure the clusters are running.

```sh
kubectl svc
```

Create the service.
```sh
kubectl create -f service.yaml
```

Deploy the images
```sh
kubectl create -f deployment.yaml
```




[link:gcp-k8s-eng]:https://console.cloud.google.com/kubernetes/