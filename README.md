# STAN'S ROBOT SHOP

This is a simple example microservices application for use as a sandbox for playing with orchestration and/or monitoring/observability techniques. It is not a reference example of how to write a microservices application, the error handling is patchy and the security is pretty much nonexistent.

See the related announcement [blog post](https://www.instana.com/blog/stans-robot-shop-sample-microservice-application/).

The application is built using these technologies:
- NodeJS ([Express](http://expressjs.com/))
- Java ([Spark Java](http://sparkjava.com/))
- Python ([Flask](http://flask.pocoo.org))
- Golang
- MongoDB
- Redis
- MySQL ([Maxmind](http://www.maxmind.com) data)
- RabbitMQ
- Nginx
- AngularJS (1.x)

The various services in this application already have any required Instana components installed and configured. These provide automatic instrumentation of the code for complete end to end [tracing](https://docs.instana.io/core_concepts/tracing/), as well as time series metrics for their runtimes.

You will need an account to see the results in the Instana dashboard. If you do not already have an account, sign up for a [free trail](https://instana.com).

## Build from Source
To build from source use Docker Compose. Optionally edit the *.env* file to specify an alternative image registry and version tag; see the official [documentation](https://docs.docker.com/compose/env-file/) for more information.

    $ docker-compose build

If you modified the *.env* file and changed the image registry, you may need to push the images to that registry

    $ docker-compose push

## Run Locally
You can run it locally for testing

    $ docker-compose up

If you are running it locally on a Linux host you can also run the Instana [agent](https://docs.instana.io/quick_start/agent_setup/container/docker/) locally, unfortunately the agent is currently not supported on Mac.

## Kubernetes
The Docker container images are all available on [Docker Hub](https://hub.docker.com/u/robotshop/). The deployment and service definition files using these images are in the *K8s* directory, use these to deploy to a Kubernetes cluster. If you pushed your own images to your registry the deployment files will need to be updated to pull from your registry; using [kompose](https://github.com/kubernetes/kompose) may be of assistance here.

If you want to deploy Stan's Robot Shop to Google Compute you will need to edit the *K8s/web-service.yaml* file and change the type from NodePort to LoadBalancer. This can also be done in the Google Compute console.

#### NOTE
I have found some issues with kompose reading the *.env* correctly, just export the variables in the shell environment to work around this.

You can also run Kubernetes locally using [minikube](https://github.com/kubernetes/minikube).

    $ kubectl create namespace robot-shop
    $ kubectl -n robot-shop create -f K8s

To deploy the Instana agent to Kubernetes, edit the *instana/instana-agent.yaml* file and insert your base64 encoded agent key. Your agent key is available from the Instana dashboard.

    $ echo -n "your agent key" | base64

Deploy the agent

    $ kubectl create -f instana/instana-agent.yaml

The agent configuration only runs the agent on nodes with the appropriate label. For minikube.

    $ kubectl label node minikube agent=instana

There is also a handy script *instana/label.sh* which labels all the nodes.

## Acessing the Store
If you are running the store locally via *docker-compose up* then, the store front is available on localhost port 8080 [http://localhost:8080](http://localhost:8080/)

If you are running the store on Kubernetes via minikube then, the store front is available on the IP address of minikube port 30080. To find the IP address of your minikube instance.

    $ minikube ip

If you are using a cloud Kubernetes / Openshift / Mesosphere then it will be available on the load balancer of that system. There will be specific blog posts on the Instana site covering these scenarios.

## Load Generation
A separate load generation utility is provided in the *load-gen* directory. This is not automatically run when the application is started. The load generator is built with Python and [Locust](https://locust.io). The *build.sh* script builds the Docker image, optionally taking *push* as the first argument to also push the image to the registry. The registry and tag settings are loaded from the *.env* file in the parent directory. The script *load-gen.sh* runs the image, edit this and set the HOST environment variable to point the load at where you are running the application. You could run this inside an orchestration system (K8s) as well if you want to, how to do this is left as an exercise for the reader.

## End User Monitoring
To enable End User Monitoring (EUM) see the official [documentation](https://docs.instana.io/products/website_monitoring/) for how to create a configuration. There is no need to inject the javascript fragment into the page, this will be handled automatically. Just make a note of the unique key and set the environment variable INSTANA_EUM_KEY for the *web* image, see *docker-compose.yaml* for an example.
