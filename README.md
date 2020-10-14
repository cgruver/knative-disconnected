# Set up Knative in a disconnected network

Clone this repository:

```bash
git clone https://github.com/cgruver/knative-disconnected-install.git
cd knative-disconnected-install
```

### Make the Knative images available to the OpenShift cluster:

The next step is to pull all of the images needed for the install, and make them available to the OpenShift cluster.

These instructions assume that you are using a personal workstation with Docker desktop.  You can also use `podman`, `skopeo`, or `buildah` to perform these steps.

The list of images is in the file `knative-images`, included with this guide.

From your internet connected workstation or bastion host:

1. Pull the images needed for the install:

    Replace the value for LOCAL_REGISTRY with the URL for your registry.

    ```bash
    export LOCAL_REGISTRY=nexus.your.domain.org:5000

    for i in $(cat knative-images)
    do 
        IMAGE_TAG=${LOCAL_REGISTRY}/$(echo $i | cut -d"/" -f2-)
        docker pull ${i}
        docker tag ${i} ${IMAGE_TAG}
    done
    ```

1. Log into your local image registry.  Since you are in a disconnected environment, you might have to change networks for this step.

    ```bash
    docker login ${LOCAL_REGISTRY}
    ```

1. Push the newly tagged images.

    ```bash
    for i in $(cat knative-images)
    do 
        IMAGE_TAG=${LOCAL_REGISTRY}/$(echo $i | cut -d"/" -f2-)
        docker push ${IMAGE_TAG}
    done
    ```

### Install Knative:

1. Create a working directory and stage the files:

    ```bash
    mkdir -p ~/knative-workdir
    cp ./*.yaml ~/knative-workdir

    cd ~/knative-workdir
    for i in $(ls *.yaml)
    do
        sed -i "s|--LOCAL_REGISTRY--|${LOCAL_REGISTRY}|g" ${i}
    done
    ```

1. Finally, deploy the Knative components:

    ```bash
    oc apply -f serverless-ns.yaml
    oc apply -f serverless-role.yaml
    oc apply -f serverless-crd.yaml
    oc apply -f serverless-operator-csv.yaml
    ```
