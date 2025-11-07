### C4: System Context

A System Context diagram illustrating the users and systems interacting with the AO-OCT Microstructure Analytics System (AO-OCT-MAS).

```mermaid
graph TD
    subgraph " "
    actor "User\n(Clinician, Researcher, Tech, Labeler, ML Engineer, Admin)" as User
    end

    subgraph External Systems
        System(Scanner / PACS)
        System(SSO Provider)
        System(Research Data Share)
    end

    subgraph "AO-OCT Microstructure Analytics System"
        System_Boundary("
        **AO-OCT-MAS**

        Ingests, processes, and analyzes AO-OCT volumes.
        Provides segmentation, metrics, and human-in-the-loop review capabilities with full data provenance.
        ")
    end

    User -- "Manages cases, reviews results, edits labels, and administers system via HTTPS" --> System_Boundary
    System_Boundary -- "Serves Web UI and API results" --> User

    System -- "Pushes raw image data via DICOM/File Drop" --> System_Boundary

    System_Boundary -- "Authenticates users via OAuth2/SAML" --> SSO_Provider
    SSO_Provider -- "Provides user identity token" --> System_Boundary

    System_Boundary -- "Exports de-identified metrics and manifests via S3/NFS" --> Research_Data_Share

    linkStyle 0,1 stroke-width:2px,fill:none,stroke:black;
    linkStyle 2 stroke-width:2px,fill:none,stroke:black;
    linkStyle 3,4 stroke-width:2px,fill:none,stroke:black;
    linkStyle 5 stroke-width:2px,fill:none,stroke:black;