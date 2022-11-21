# java-pipeline

An exemplar Java + Maven + Spring Boot project with Jenkins pipeline using Maven lifecycle and Docker for packaging and running integration tests

This exemplar configuration includes:

- Packaging as a Docker image.
- Pipeline as code with Jenkins.
- Surefire configured to gather test coverage with JaCoCo.
- Mutation tests with Pitest.
- Integration tests with Selenium, which can be executed either manually or via Failsafe.
- Integration test coverage with JaCoCo.
- Load tests with JMeter.
- Dependency vulnerability scan with OWASP Dependency Check.
- Quality analysis with SonarQube, including collating results from the other tools.

## Set up in Jenkins

The continuous integration pipeline requires that a credential with id `Praveen-docker-hub`
is configured in Jenkins. The `Praveen` prefix in the credential id refers to the `Praveen`
org namespace which is targeted to push Docker images to Docker Hub.

If you want to use your own Docker Hub organization, edit the pipeline replacing the `ORG_NAME` variable with the chosen organization name, and configure the credential in Jenkins with id `<YOUR_ORG_NAME-docker-hub>`.

## Build and test locally

To build and launch the project, to ensure that everything works fine before commiting any change, just leverage the usual Maven commands:

    mvnw verify
    mvnw spring-boot:run

Once up and running, access the following URLs to get status information, a generic greeting message, and a personalized greeting message:

    http://localhost:8080/actuator/health
    http://localhost:8080/hello
    http://localhost:8080/hello/John

## Build and test locally with k3s (Rancher Desktop)

There are many ways to have a Kubernetes cluster available for development purposes, but possibly one of the simplest and fastest is to install Rancher Desktop in your workstation. With Rancher Desktop comes k3s, a lightweight Kubernetes distribution optimized to be used in a workstation and other resource-limited environments.

Once Rancher Desktop is installed and running, we can use `nerdctl` command line tool to build images and `kubectl` to run them.

To build the image, first build the project with Maven as usual and afterwards, execute the image build command:

    mvnw verify
    nerdctl --namespace k8s.io build -t Praveen-demos-java-pipeline:1.0-SNAPSHOT .

The namespace parameter is needed for the image to be available to the local Kubernetes cluster. Once ready, launch the service with this command:

    kubectl run javapipeline --image Praveen-demos-java-pipeline:1.0-SNAPSHOT

To simplify local testing, instead of exposing a service or configuring an ingress, let's simply enable port forwarding:

    kubectl port-forward pods/javapipeline 8080:8080

Meanwhile the port forward is active (finish it pressing Ctrl+C in the terminal where the process is running), the service will be available through `localhost` as if it was running as a local process:

    http://localhost:8080/actuator/health
    http://localhost:8080/hello
    http://localhost:8080/hello/John

Once the local tests have finished, terminate the service with the following command:

    kubectl delete pod javapipeline
