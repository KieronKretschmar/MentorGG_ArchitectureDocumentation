# Design

MentorEngine is a set of services running on a [Kubernetes](https://kubernetes.io/) cluster that make up the MENTOR.GG backend.


## AMQP vs REST
- AMQP is used for internal asynchronous communication, particularly where messages a) might queue up and/or b) should not get lost in the event of a service being unavailable.
- REST is used for synchronous communication, particularly where a) the requesting service requires a synchronous response and/or b) the request comes from outside MentorEngine.

Architecture design documentation


## Resources:

- [Markdown Guide](https://about.gitlab.com/handbook/product/technical-writing/markdown-guide/)
- [RabbitMQ on GCP](https://github.com/GoogleCloudPlatform/click-to-deploy/blob/master/k8s/rabbitmq/README.md)
- [Design Patterns for Microservices](https://dzone.com/articles/design-patterns-for-microservices)
