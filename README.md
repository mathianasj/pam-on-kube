# PAM in Containers

## Decision Manager

### S2I

Demonstrates the cababilities of running an immutable image for DM inside of a container using the S2I build process.  We will use one of the example GIT repositiories for a simple drl rules project.

#### Resources

|Title|URL|
|-----|---|
|S2I builder image|https://github.com/jboss-container-images/rhdm-7-openshift-image
|Example Rules Project|https://github.com/jboss-container-images/rhdm-7-openshift-image/blob/main/quickstarts/hello-rules/README.md

#### Walkthrough
1. Run the S2I build against the hello rules sample repo
    ```
    s2i build https://github.com/jboss-container-images/rhdm-7-openshift-image.git --context-dir quickstarts/hello-rules/hellorules registry.redhat.io/rhdm-7/rhdm-kieserver-rhel8:7.10.1 quay.io/{REPO_HERE}/testdmn:demo
    ```
1. Push the built image into a repsoitory which the kubernetes cluster will have access
    ```
    docker push quay.io/mathianasj/testdmn:demo
    ```
1. Deploy the kubernetes resources
    1. You will need to update ingress.yaml host to match your deployment's ingress settings
    ```
    kubectl create -f ./deploy/deployment.yaml
    kubectl create -f ./deploy/service.yaml
    kubectl create -f ./deploy/ingress.yaml
    ```
1. Build the client and test the execution
    1. Clone the repo locally
        ```
### S2I
    1. Build the client app with maven
        ```
        mvn clean package install
        cd hellorules-client
        ```
    1. Run the client and execute against the kieserver running in kubernetes
        ```
        mvn clean compile exec:java \
        -DskipTests \
        -Dexec.mainClass="org.openshift.quickstarts.rhdm.kieserver.hellorules.client.HelloRulesClient" \
        -Dexec.args="runRemoteRest" \
        -Dusername=adminUser \
        -Dpassword="admin1!" \
        -Dhost="testdmn.example.com" \
        -Dport=80
        ```


## Process Automation Manager

### S2I

#### Resources

|Title|URL|
|-----|---|
|S2I builder image|https://github.com/jboss-container-images/rhpam-7-openshift-image
|Example Rules Project|https://github.com/jboss-container-images/rhpam-7-openshift-image/blob/main/quickstarts/hello-rules/README.md

#### Walkthrough
1. Run the S2I build against the sample library process quickstart
    ```
    s2i build https://github.com/jboss-container-images/rhpam-7-openshift-image.git --context-dir quickstarts/library-process/library registry.redhat.io/rhpam-7/rhpam-kieserver-rhel8:7.10.1 quay.io/{REPO_HERE}/testpam:demo
    ```
1. Push the built image into a repsoitory which the kubernetes cluster will have access
    ```
    docker push quay.io/mathianasj/testpam:demo
    ```
1. Deploy the kubernetes resources
    1. You will need to update ingress.yaml host to match your deployment's ingress settings
    ```
    kubectl create -f ./pam/deployment.yaml
    kubectl create -f ./pam/service.yaml
    kubectl create -f ./pam/ingress.yaml
    ```
1. Build the client and test the execution
    1. Build the client app with eap s2i
        ```
        s2i build https://github.com/jboss-container-images/rhpam-7-openshift-image.git registry.redhat.io/jboss-eap-7/eap73-openjdk8-openshift-rhel7:latest quay.io/mathianasj/testpamclient:demo --context-dir quickstarts/library-process
        ```
    1. Push the built image into a repsoitory which the kubernetes cluster will have access
        ```
        docker push quay.io/mathianasj/testpamclient:demo
        ```
    1. You will need to update ingress.yaml host to match your deployment's ingress settings
        ```
        kubectl create -f ./pam/client_deployment.yaml
        kubectl create -f ./pam/client_service.yaml
        kubectl create -f ./pam/client_ingress.yaml
        ```
    1. Run the client and execute against the kieserver running in kubernetes
        ```
        curl http://testpamclient.example.com/library?command=runRemoteRest&host=testpam&port=8080&username=adminUser&password=admin1!
        ```
