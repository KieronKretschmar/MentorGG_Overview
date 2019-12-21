# Overview
Below is an overview over the MENTOR.GG code repositories and service structure. 

See [Design](https://gitlab.com/mentorgg/documentation/design) and [Implementation](https://gitlab.com/mentorgg/documentation/implementation) for further documentation.

## Service Outline
- **Frontend**
    - [**Vue-WebApp**](https://gitlab.com/mentorgg/Frontend/mentor-gg-WebApp)
        The MENTOR.GG Vue app.
- **Infrastructure**
    - [**MentorInterface**](https://gitlab.com/mentorgg/engine/mentor-interface)
        REST API exposed to the internet via an Ingress, providing authentication services and access to the Mentor Engine, and aggregates data from different sources.
    - [**RabbitMQCluster**](https://gitlab.com/mentorgg/engine/rabbitmqcluster)
        Self-hosted RabbitMQ Cluster for internal AMQP queues between services.
- **CS:GO**:
    - [**DemoCentral**](https://gitlab.com/mentorgg/csgo/democentral)
        Orchestrate demo acquisition and analysis.
    - [**DemoDownloader**](https://gitlab.com/mentorgg/csgo/demodownloader)
        Download demos either from URL or file stream.
    - [**DemoFileWorker**](https://gitlab.com/mentorgg/csgo/demofileworker)
        Obtain raw match data from a demo file and enriches the result.
    - [**MatchDBI**](https://gitlab.com/mentorgg/csgo/matchdbi)
        Store and retrieve match data.
    - [**SituationOperator**](https://gitlab.com/mentorgg/csgo/situationsoperator)
        Store, retrieve and compute situation data, e.g. misplays.
    - [**FaceitMatchGatherer**](https://gitlab.com/mentorgg/csgo/faceitmatchgatherer)
        Poll Faceit API for new matches.
    - [**SharingCodeGatherer**](https://gitlab.com/mentorgg/csgo/sharingcodegatherer)
        Poll Steam SharingCode API for new SharingCodes.
    - [**SteamworksService**](https://gitlab.com/mentorgg/csgo/steamworksservice)
       Translates SharingCodes into demo download urls.
    - [**ConfigurationDBI**](https://gitlab.com/mentorgg/csgo/configurationdbi)
        Provide configuration data to other services (e.g. Equipment, Ingame2Px conversion parameters).
    - [**SteamUserProjects**](https://gitlab.com/mentorgg/engine/steamuserprojects)
        Provide info about steam users.
    - [**MatchEntities**](https://gitlab.com/mentorgg/csgo/matchentities)
        Classes for data extracted from demos referenced by multiple projects.

## Information Flow

```mermaid
graph TD;
    I["🌎"] --- MI
    MI --- UDB((UserDB));
    
    MI --- SUO[SteamUserOperator];
    
    MI --- SCG["SharingCodeGatherer 💾"];
    SCG -.- SWS[SteamworksService];
    SWS -.- DC;
    
    MI --- FG["FaceitMatchGatherer 💾"];
    FG -.- DC;
    
    
    MI --- CDBI["ConfigurationDBI 💾"];
    
    MI[MentorInterface] --- DC["DemoCentral 💾"];
    DC -.- DD[DemoDownloader];
    DC -.- DFW[DemoFileWorker];
    DFW -.- MEF["MatchEntities Fanout"];
    MEF -.- MDBI;
    MEF -.- SO;
    
    CDBI --- DFW;

    MI --- MDBI["MatchDBI 💾"];
    MI --- SO["SituationOperator 💾"];

    RC["🐰 RabbitMQCluster"];


    classDef db fill:white;
    classDef queue fill:pink;
    class RFO queue;
    class UDB,SUDB,CDB,SDB,MDB db;

    click MI,UDB "https://gitlab.com/mentorgg/engine/mentor-interface";
    click SUO "https://gitlab.com/mentorgg/engine/steamuserprojects";
    click SCG "https://gitlab.com/mentorgg/csgo/sharingcodegatherer";
    click SWS "https://gitlab.com/mentorgg/csgo/steamworksservice";
    click CDBI,CDB "https://gitlab.com/mentorgg/csgo/configurationdbi";
    click DC "https://gitlab.com/mentorgg/csgo/democentral";
    click FG "https://gitlab.com/mentorgg/csgo/faceitmatchgatherer";
    click DFW "https://gitlab.com/mentorgg/csgo/demofileworker";
    click DD "https://gitlab.com/mentorgg/csgo/demodownloader";
    click MDBI,MDB "https://gitlab.com/mentorgg/csgo/matchdbi";
    click RC "https://gitlab.com/mentorgg/engine/rabbitmqcluster";
    click MEF "https://gitlab.com/mentorgg/csgo/matchentities";
    click SO,SDB "https://gitlab.com/mentorgg/csgo/situationsoperator";
```




```mermaid
graph TD;
    subgraph Legend
        AMQPP["AMQP Producer"] -.- AMQPC["AMQP Consumer"];
        HTTPC["HTTP Client"] --- HTTPS["HTTP Server"];
        SWDB["Service with attached DB 💾"]
    end
```
