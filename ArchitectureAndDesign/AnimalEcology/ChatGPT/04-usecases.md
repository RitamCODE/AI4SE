# Use Case Diagram

```plantuml
@startuml
title Edge-to-Cloud Wildlife Intrusion Response — Use Cases

left to right direction

actor "Field Technician" as FieldTechnician
actor "Drone Operator" as DroneOperator
actor "Wildlife Scientist" as WildlifeScientist
actor "Data/ML Engineer" as DataMLEngineer
actor "DevOps / Platform Admin" as DevOpsAdmin
actor "Time" as TimeTrigger

usecase "Configure Edge Device\n(US-01, US-04)" as UC_ConfigEdge
usecase "View Edge Health\n(US-02)" as UC_ViewHealth
usecase "Continuous Capture & Scoring\n(US-16, US-17)" as UC_CaptureScore
usecase "Filter & Approve Intrusion Alerts\n(US-18, US-19)" as UC_FilterAlerts
usecase "Plan & Approve Drone Mission\n(US-10–US-15)" as UC_PlanMission
usecase "Execute Drone Mission & Capture Media\n(US-10–US-15)" as UC_ExecuteMission
usecase "Reliable Media Transfer to OSC\n(US-24–US-28)" as UC_TransferMedia
usecase "Launch & Monitor Tapis Jobs\n(US-29–US-34)" as UC_ManageJobs
usecase "Manage AOIs / Thresholds / Models\n(US-04–US-09, US-35)" as UC_ManageConfig
usecase "Monitor Metrics & Alerts\n(US-36–US-38)" as UC_Monitor

FieldTechnician --> UC_ConfigEdge
FieldTechnician --> UC_ViewHealth

WildlifeScientist --> UC_ManageConfig

DroneOperator --> UC_PlanMission
DroneOperator --> UC_ExecuteMission

DataMLEngineer --> UC_ManageJobs

DevOpsAdmin --> UC_ManageConfig
DevOpsAdmin --> UC_Monitor

TimeTrigger --> UC_CaptureScore

UC_CaptureScore --> UC_FilterAlerts
UC_FilterAlerts --> UC_PlanMission
UC_PlanMission --> UC_ExecuteMission
UC_ExecuteMission --> UC_TransferMedia
UC_TransferMedia --> UC_ManageJobs
UC_ManageJobs --> UC_Monitor
@enduml
