# PAM in Containers

## DMN

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
    s2i build https://github.com/jboss-container-images/rhdm-7-openshift-image.git --context-dir quickstarts/hello-rules/hellorules registry.redhat.io/rhdm-7/rhdm-kieserver-rhel8:7.10.1 quay.io/mathianasj/testdmn:demo
    ```
1. Push the built image into a repsoitory which the kubernetes cluster will have access
    ```
    docker push quay.io/mathianasj/testdmn:demo
    ```
1. Deploy the kubernetes resources
    1. You will need to update ingress.yaml host to match your deployment's ingress settings
    ```
    kubectl create -f ./deploy/deployment.yaml
    kubectl create -f ./deploy/ingress.yaml
    ```
1. Build the client and test the execution
    1. Clone the repo locally
        ```
        git clone https://github.com/jboss-container-images/rhdm-7-openshift-image.git
        cd rhdm-7-openshift-image/quickstarts/hello-rules
        ```
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