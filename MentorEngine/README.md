# MentorEngine

MentorEngine is a set of services running on a [Kubernetes][K8] cluster that make up the MENTOR.GG backend.

## Service Outline

- **MentorInterface**
    REST API exposed to the internet via an Ingress, providing authentication services and access to the Mentor Engine, and aggregates data from different sources.
- CS:GO:
    - **DemoCentral**
        Orchestrate demo acquisition and analysis.
    - **DemoDownloader**
        Download demos either from URL or file stream.
    - **DemoFileWorker**
        Obtain raw match data from a demo file and enriches the result.
    - **MatchDBI**
        Store and retrieve match data.
    - **SituationOperator**
        Store, retrieve and compute situation data, e.g. misplays.
    - **FaceitMatchGatherer**
        Poll Faceit API for new matches.
    - **SharingCodeGatherer** 
        Poll Steam SharingCode API for new matches.
    - **ConfigurationDBI**
        Provide configuration data to other services (e.g. Equipment, Ingame2Px conversion parameters).
    - **SteamUserProjects**
        Provide info about steam users.

## Information Flow

```mermaid
graph TD;
    I["🌎"] --- MI
    MI --- UDB((UserDB));
    
    MI --- SUDBI(SteamUserDBI);
    SUDBI --- SUDB((SteamUserDB));
    SUDBI --- SUDG[SteamUserDataGatherer];
    
    MI --- SCO[SharingCodeGatherer];
    SCO --- SWC[SteamworksConnection];
    SCO -.- DC;
    
    MI --- FG[FaceitMatchGatherer];
    FG -.- DC;
    
    
    MI --- CDBI[ConfigurationDBI];
    
    MI[MentorInterface] --- DC[DemoCentral];
    DC -.- DD[DemoDownloader];
    DC -.- DFW[DemoFileWorker];
    DFW -.- RFO["🐰 Fanout"];
    RFO -.- MDBI;
    RFO -.- SO;
    
    CDBI --- DFW;

    CDBI --- CDB((ConfigurationDB));
    MI --- MDBI[MatchDBI];
    MI --- SO[SituationOperator];
    SO --- SDB((SituationDB));
    MDBI --- MDB((MatchDB));

    RC["🐰 RabbitMQCluster"];

    classDef db fill:white;
    classDef queue fill:pink;
    class RFO queue;
    class UDB,SUDB,CDB,SDB,MDB db;

    click MI,UDB "https://gitlab.com/mentorgg/engine/mentor-interface";
    click SUDBI,SUDB,SUDG "https://gitlab.com/mentorgg/engine/steamuserprojects";
    click SCO,SWC "https://gitlab.com/mentorgg/csgo/sharingcodeprojects";
    click CDBI,CDB "https://gitlab.com/mentorgg/csgo/configurationdbi";
    click DC "https://gitlab.com/mentorgg/csgo/democentral";
    click FG "https://gitlab.com/mentorgg/csgo/faceitmatchgatherer";
    click DFW "https://gitlab.com/mentorgg/csgo/demofileworker";
    click DD "https://gitlab.com/mentorgg/csgo/demodownloader";
    click MDBI,MDB "https://gitlab.com/mentorgg/csgo/matchdbi";
    click RC,RFO "https://gitlab.com/mentorgg/engine/rabbitmqcluster";
    click SO,SDB "https://gitlab.com/mentorgg/csgo/situationsoperator"


    
```

## Publishing
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
