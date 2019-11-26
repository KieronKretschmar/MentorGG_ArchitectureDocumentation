# MentorEngine

MentorEngine is a set of services running on a [Kubernetes][K8] cluster that make up the MENTOR.GG backend.


## AMQP vs REST
- AMQP is used for internal asynchronous communication, particularly where messages a) might queue up and/or b) should not get lost in the event of a service being unavailable.
- REST is used for synchronous communication, particularly where a) the requesting service requires a synchronous response and/or b) the request comes from outside MentorEngine.

## Publishing process
- Updating MatchEntities
    - MatchEntities is referenced by multiple projects, as it holds the definition for the MatchDataSets being transferred to MatchDBI and more, and also is the Code First basis for the MatchDB schema.
    - Consumers of MatchDataSets (MatchDBI, SituationOperator) should only accept MatchDataSets of one major version.
    

    - Publishing major updates (breaking changes) to MatchEntities should follow this pattern:
        1. Update DemoFileWorker so that it publishes MatchDataSets of newer version.
        2. Wait until all MatchDataSets of the previous version are consumed.
        3. Update consumers (including MatchDBI and therefore the MatchDB using migrations) so that they only accept MatchDataSets of the newer version.
        
        If, for some reason, MatchDataSets of the older version appear at consumers after Step 3, the entire analysis of the match should be restarted beginning with (the new) DemoFileWorker.

## Further Reading:

- [RabbitMQ on GCP](https://github.com/GoogleCloudPlatform/click-to-deploy/blob/master/k8s/rabbitmq/README.md)
- [Design Patterns for Microservices](https://dzone.com/articles/design-patterns-for-microservices)

[K8]: https://kubernetes.io/
