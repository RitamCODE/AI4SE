# Design: Sequence Diagrams

## 1. Event Detection and Scoring
```mermaid
sequenceDiagram
    participant CT as Camera Trap
    participant EP as Event Processor
    CT->>EP: New Image (with metadata)
    EP->>CT: Acknowledge
    CT->>CT: Score Image (TensorFlow Lite)
    CT->>EP: Scored Event

## 2. Drone Mission Planning

sequenceDiagram
    participant EP as Event Processor
    participant FP as Flight Planner
    participant DM as Drone Manager
    EP->>FP: New Event (AOI, priority)
    FP->>DM: Request Flight Approval
    DM-->>FP: Approval Status
    FP->>DM: Flight Plan (GeoJSON)
