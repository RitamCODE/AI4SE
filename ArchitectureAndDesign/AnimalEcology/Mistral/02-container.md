
---

### **2. `02-container.md`**
```markdown
# C4: Container

## Diagram (Mermaid)
```mermaid
graph TD
    subgraph System
        A[Camera Trap Controller] -->|HTTP/REST| B[Event Processor]
        B -->|gRPC| C[Flight Planner]
        C -->|MQTT| D[Drone Manager]
        D -->|SFTP| E[Data Transfer Agent]
        E -->|Tapis API| F[OSC Job Orchestrator]
        B -->|AMQP| G[Alert Service]
    end

Container,Purpose,Tech Hint,Responsibilities
Camera Trap Controller,On-device detection and scoring,Python, TensorFlow Lite,Runs object detection, scores images, and forwards events.
Event Processor,Filters and coalesces events,Node.js,Applies business rules, reduces noise, and triggers downstream actions.
Flight Planner,Plans drone missions over AOIs,Python, GeoPandas,Generates flight paths, checks constraints, and submits for approval.
Drone Manager,Manages drone operations,Python, MAVLink,Executes flights, monitors status, and handles RTH.
Data Transfer Agent,Transfers media to OSC,Go, Tapis SDK,Ensures reliable, resumable, and checksum-verified data transfer.
OSC Job Orchestrator,Submits jobs to OSC via Tapis,Python, Tapis CLI,Triggers training/inference jobs and monitors their status.
Alert Service,Notifies stakeholders,Node.js, Twilio,Sends alerts for events, failures, and job completions.
