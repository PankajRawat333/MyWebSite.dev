Title: Run Azure Functions in a Docker Container
Lead: Run Azure Function in a Docker container and run anywhere!
Description: We can run Azure Function in a Docker container then we can run it any cloud or On Premises environment.
Published: 22/07/2019
Image: /posts/images/azure-function-docker.jpg
PrimaryTag: azure
Tags:
  - azure
  - function
  - docker
  - container
---
As you might already aware that .NET Core is a cross-platform applications targeting Windows, Linux and macOS and Docker gives more flexibility to run .Net Core application anywhere any devices. Latest Azure function version 2 runs on .NET Core, which means it is cross-platform. Anyone wants to run Azure function on Linux or small non-windows devices can easily run same Azure functions.

Docker provides more flexibility to run Azure functions anywhere, you can take Azure function base image and build your docker image, once your image is ready you can run your container anywhere. You can run on-premise or Other cloud providers. Microsoft also using Docker container to ship IoT edge module or Off-line cognitive services on IoT edge solutions.

### Serverless Azure Functions vs Azure Functions in container

Microsoft design Azure function as Serverless, so the developer doesn't need to focus to setup infrastructure and scaling requirement. Another side when you will create docker container you need to manage your container instance (using Kubernetes or Docker swarm) and take care of all scaling requirement while Azure function will manage by Azure and can scale limitless.

### Where you should run Azure Functions without a container

- When you are building Azure function that will run only on Azure Cloud
- When you want to go with true serverless (don't want to take care of infrastructure)
- Limitless scale

### Where you should run Azure Functions in a docker container

- When you want to run Azure function Anywhere (On-premise or Other cloud vendors)
- When you want to run Azure function on IoT Edge Devices
- Don't need limitless scale and already running/managing other services through docker container.

[KEDA](https://github.com/kedacore/keda) allows for fine-grained autoscaling (including to/from zero) for event-driven Kubernetes workloads. KEDA serves as a Kubernetes Metrics Server and allows users to define autoscaling rules using a dedicated Kubernetes custom resource definition.

### References

[Azure Functions on Kubernetes with KEDA](https://learn.microsoft.com/en-us/azure/azure-functions/functions-kubernetes-keda)

Happy cloud computing.