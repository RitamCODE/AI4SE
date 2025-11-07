# Use Case Diagram

## Diagram (Mermaid)
```mermaid
graph TD
    A[Field Operations] -->|Deploy Camera Traps| B[Camera Trap Controller]
    A -->|Monitor System| B
    C[Wildlife Scientists] -->|Configure AOIs| B
    D[Drone Operators] -->|Approve Flights| E[Flight Planner]
    B -->|Trigger Event| F[Event Processor]
    F -->|Plan Flight| E
    E -->|Execute Flight| G[Drone Manager]
    G -->|Collect Data| H[Data Transfer Agent]
    H -->|Transfer Data| I[OSC Job Orchestrator]
    I -->|Trigger Job| J[OSC/Tapis]
