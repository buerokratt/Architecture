### Prepare training data for training

> Participant `Trigger` can be initiated via `GUI`, `CronManager`, or other similar

```mermaid
sequenceDiagram

participant Trigger
participant Ruuter
participant Resql
participant DataMapper
participant CronManager
participant Postgres

autonumber
Trigger->>Ruuter: [POST] /training/model/prepare

Ruuter->>+Resql: [GET] /training/models/versions/latest
Resql->>+Postgres: [Fetch latest model version nr]
Postgres-->>-Resql: [v1.17]
Resql-->>-Ruuter: [v1.17]

Ruuter->>+Resql: [POST] /training/models/versions/increase<br>[v1.18]
Resql->>+Postgres: [v.18] [UNPROCESSED]
Postgres-->>-Resql: [OK]
Resql-->>-Ruuter: [OK]

Ruuter->>CronManager: [POST] /training/initiate/v1.18
CronManager->>+DataMapper: [POST] /training/initiate/v1.18
DataMapper->>DataMapper: Merge unprocessed YAML files
DataMapper-->>-CronManager: [v1.18.tar.gz]

CronManager->>+Ruuter: [v1.18] [v1.18.tar.gz] [PROCESSING]
Ruuter->>+Resql: [v1.18] [v1.18.tar.gz] [PROCESSING]
Resql->>+Postgres: [v1.18] [v1.18.tar.gz] [PROCESSING]
Postgres-->>-Resql: [OK]
Resql-->>-Ruuter: [OK]

loop Check if all files have been processed
    CronManager->>CronManager: Workind folder must be empty
    note over CronManager,CronManager: If folder is empty
    CronManager->>Ruuter: [POST] /training/models/status [v1.18] [v1.18.tar.gz] [UNTRAINED]
    
    Ruuter->>+Resql: [POST] /training/models/status [v1.18] [v1.18.tar.gz] [UNTRAINED]
    Resql->>+Postgres: [v1.18] [v1.18.tar.gz] [UNTRAINED]
    Postgres-->>-Resql: [OK]
    Resql-->>-Ruuter: [OK]
end
```

<br><br><br><br>

### Check if there are any models to be trained

```mermaid
sequenceDiagram

participant Ruuter
participant Resql
participant CronManager
participant Postgres
participant Rasa

autonumber
loop Check if there are new models to be trained
    CronManager->>+Ruuter: [GET] /training/models/untrained
    
    Ruuter->>+Resql: [GET] /training/models/untrained
    Resql->>+Postgres: [UNTRAINED]
    Postgres-->>-Resql: [v1.18] [v1.18.tar.gz]
    Resql-->>-Ruuter: [v1.18] [v1.18.tar.gz]

    Ruuter-->>-CronManager: [v1.18] [v1.18.tar.gz]

    CronManager->>Rasa: [v1.18] [v1.18.tar.gz]
end
```

<br><br><br><br>

### Mark the training process as final

```mermaid
sequenceDiagram

participant Ruuter
participant Resql
participant Postgres
participant Rasa

autonumber
Rasa->>Ruuter: [POST] /training/rasa/models/status [v1.18] [v1.18.tar.gz] [TRAINED]

Ruuter->>+Resql: [POST] /training/rasa/models/status [v1.18] [v1.18.tar.gz] [TRAINED]
Resql->>+Postgres: [v1.18] [v1.18.tar.gz] [TRAINED]
Postgres-->>-Resql: [OK]
Resql-->>-Ruuter: [OK]
```

<br><br><br><br>

### Deployment of newly trained model

_TBA_
