# Use Case Diagram

## Use Cases - Wildlife Intrusion Response System
````mermaid
graph TD
    subgraph Actors
        FT[Field Technician]
        DO[Drone Operator]
        WS[Wildlife Scientist]
        MLE[ML Engineer]
        PA[Platform Admin]
    end

    subgraph "Use Cases"
        UC1[Setup Edge Device<br/>US-01]
        UC2[Monitor Device Health<br/>US-02,US-07]
        UC3[Configure Ring Buffer<br/>US-04]
        UC4[Update Edge Software<br/>US-06]
        
        UC5[Approve Mission<br/>US-08]
        UC6[Define Auto-Launch Policy<br/>US-09]
        UC7[Pause/Abort Mission<br/>US-12]
        
        UC8[Define AOI<br/>US-14]
        UC9[Set Species Thresholds<br/>US-15]
        UC10[Review Events & Media<br/>US-17,US-19]
        UC11[Provide Label Feedback<br/>US-18]
        
        UC12[Deploy Model Version<br/>US-20]
        UC13[Configure Tapis Job<br/>US-22]
        UC14[Monitor Drift<br/>US-23]
        
        UC15[Manage RBAC<br/>US-26]
        UC16[Rotate Credentials<br/>US-27]
        UC17[View Observability<br/>US-28]
        
        UC18[Capture & Score Frame<br/>FR-001,FR-002]
        UC19[Filter & Coalesce Events<br/>FR-006,FR-007,US-36]
        UC20[Generate Flight Plan<br/>FR-010,FR-011]
        UC21[Execute Mission<br/>FR-014,FR-016]
        UC22[Transfer Data to OSC<br/>FR-019,FR-020,US-37]
        UC23[Submit Tapis Job<br/>FR-023,FR-024]
    end

    FT --> UC1
    FT --> UC2
    FT --> UC3
    FT --> UC4
    
    DO --> UC5
    DO --> UC6
    DO --> UC7
    
    WS --> UC8
    WS --> UC9
    WS --> UC10
    WS --> UC11
    
    MLE --> UC12
    MLE --> UC13
    MLE --> UC14
    
    PA --> UC15
    PA --> UC16
    PA --> UC17
    
    UC18 -.includes.-> UC19
    UC19 -.triggers.-> UC20
    UC20 -.triggers.-> UC21
    UC21 -.includes.-> UC22
    UC22 -.triggers.-> UC23
    
    UC5 -.extends.-> UC20
    UC7 -.extends.-> UC21
````

Caption:
Use cases grouped by actor role. System-level flows (UC18-23) show the core pipeline from frame capture to Tapis job submission. Operator approval (UC5) extends flight planning; pause/abort (UC7) extends mission execution.