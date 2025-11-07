## Diagram (Mermaid)
```mermaid
graph TD
    subgraph System Boundary
        A[Wildlife Intrusion Response System]
    end
    B[Field Operations] --> Deploy/Monitor| A
    C[Wildlife Scientists] -->|Configure AOIs| A
    D[Drone Operators] -->|Approve Flights| A
    E[OSC/Tapis] -->|Trigger Jobs| A
    F[Camera Traps] -->|Capture Media| A
    G[Drones] -->|Collect Data| A
    A -->|Alerts/Reports| B
    A -->|Data/Logs| E