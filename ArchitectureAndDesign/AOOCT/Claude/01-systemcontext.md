# C4: System Context

```mermaid
C4Context
    title System Context - AO-OCT Microstructure Analytics System

    Person(researcher, "Researcher/Clinician", "Analyzes retinal microstructures")
    Person(tech, "Imaging Technician", "Monitors pipeline runs")
    Person(labeler, "Labeler/Adjudicator", "Reviews and edits masks")
    Person(mlengineer, "ML Engineer", "Trains and promotes models")
    Person(admin, "Admin/Auditor", "Manages access and compliance")
    
    System(aomas, "AO-OCT-MAS", "Ingests volumes, corrects artifacts, segments microstructures, computes metrics")
    
    System_Ext(sso, "SSO Provider", "Authentication")
    System_Ext(scanner, "AO-OCT Scanner/PACS", "Source of imaging data")
    System_Ext(storage, "Institutional Storage", "Research data exports")
    System_Ext(dicom, "DICOM Network", "Optional C-STORE listener")
    
    Rel(researcher, aomas, "Uploads volumes, views metrics/overlays", "HTTPS/REST")
    Rel(tech, aomas, "Monitors runs, re-queues failures", "HTTPS/REST")
    Rel(labeler, aomas, "Edits masks, reviews active-learning queue", "HTTPS/REST")
    Rel(mlengineer, aomas, "Registers models, manages deployments", "HTTPS/REST")
    Rel(admin, aomas, "Manages RBAC, audits access", "HTTPS/REST")
    
    Rel(aomas, sso, "Authenticates users", "SAML/OAuth2")
    Rel(scanner, aomas, "Pushes volumes", "DICOM C-STORE/S3/NFS")
    Rel(dicom, aomas, "Sends DICOM studies", "DICOM")
    Rel(aomas, storage, "Exports metrics/manifests", "S3/NFS")
    
    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="2")
```

**Caption**: External actors interact with AO-OCT-MAS via REST APIs. System integrates with SSO for auth, receives data from scanners/PACS, and exports to institutional storage. All PHI boundaries enforce encryption and access control.