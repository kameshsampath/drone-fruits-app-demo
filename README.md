# Fruits Demo

A Drone pipeline JAR(Java And Reactive) stack to show build, test and deploy a [Java](https://jdk.java.net/) API with [React](https://reactjs.org/) Frontend with optional DB.

The stack has the following components,

- A [Quarkus](https://quarkus.io) based Java REST API with optional persistence using Hibernate
- An React based UI application

## Pre-requisites

- [Docker Desktop for Mac/Windows](https://www.docker.com/products/docker-desktop/) and Docker on Linux

- Drone Desktop Extension,

  ```shell
   docker volume create drone-desktop-data
   docker extension install kameshsampath/drone-desktop-extension:v1.1.2
  ```

## Clone Sources

```shell
git clone htps://github.com/kameshsampath/drone-fruits-app-demo
cd drone-fruits-app-demo
export DEMO_HOME="$PWD"
```

__NOTE__: The demo sources that we cloned will be referred to as `$DEMO_HOME`

## Set Environment

### Secret File

Create the secret file

```shell
cp secret.example my.secret
```

Edit the `my.secret` and update the value to suit your environment and settings.

```shell
export SECRET_FILE=my.secret
```

### Environment file

Create the secret file

```shell
cp .env.example .env
```

Edit the `.env` and update the value to suit your environment and settings.

```shell
export ENV_FILE=.env
```

## Build and Deploy

We can use kind as target Kubernetes cluster that can be used by our pipeline,

Start Kind Cluster

```shell
$DEMO_HOME/hack/kind.sh
```

We need `kubeconfig` that is routable via docker network `kind` to push images to the registry.

Run the following command to retrieve the `kubeconfig`,

```shell
kind get kubeconfig --internal --name=ccd-demo-cluster > .kube/config.internal
```

```shell
export DOCKER_NETWORK=kind
ln -s $DEMO_HOME/.drone.kind.yml .drone.yml
```

As local docker registry will be used to push the images. Refer to [Kind document](https://kind.sigs.k8s.io/docs/user/local-registry/) for more details.

Update the secrets file `my.secret` for API and UI image name as shown below:

```shell
api_image=kind-registry:5000/example/fruits-api
ui_image=kind-registry:5000/example/fruits-ui
```

Run the pipeline

```shell
drone exec --env-file="${ENV_FILE}" --secret-file="${SECRET_FILE}" --network="${DOCKER_NETWORK}" --trusted
```

__NOTE__: The drone commands are given for understanding, when using Drone Desktop Docker extension it takes care of running these commmands for you.

## Deploy Google Cloud

### Pre-requisites

- [Google Cloud](https://cloud.google.com/) Service Account(SA) with permissions to,
  - Ability to deploy to [Google Cloud Run](https://cloud.google.com/run)
  - Ability to push to [Google Cloud Registry](https://cloud.google.com/container-registry/)
  
- Optionally [gcloud CLI](https://cloud.google.com/cli)

This section will demonstrate on how to use the drone pipeline to deploy the API on to Google Cloud Run and the frontend UI to [vercel](https://vercel.com)

__WARNING__: The following section is __WIP__

```shell
export DOCKER_NETWORK=kind
ln -s $DEMO_HOME/.drone.gcloud.yml .drone.yml
```

__IMPORTANT__: Make sure you have updated all the GCP related parameters in your secret before you run the pipeline

Run the pipeline

```shell
drone exec --env-file="${ENV_FILE}" --secret-file="${SECRET_FILE}" --trusted
```

### API Access

#### Make Service publicly accessible

As the API is not enabled with authentication by default to quickly test the application try allowing `allUsers` to access the API,

```shell
gcloud run services add-iam-policy-binding fruits-api \
  --region="${GCP_REGION}" \
  --member="allUsers" \
  --role="roles/run.invoker"
```

#### Disable Public Access to API

To switch back to authentication mode use the following command,

```shell
gcloud run services remove-iam-policy-binding fruits-api --member='allUsers' --role='roles/run.invoker' --region="${GCP_REGION}"
```
