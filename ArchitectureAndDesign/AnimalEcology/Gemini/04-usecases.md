### Use Case Diagram

```plantuml
@startuml
title Use Case Diagram - Wildlife Intrusion Response

left to right direction

actor "Field Technician" as fieldTech
actor "Drone Operator" as droneOp
actor "Wildlife Scientist" as scientist
actor "Data/ML Engineer" as mlEng
actor "DevOps/Platform Admin" as admin

rectangle "Wildlife Intrusion Response System" {
  usecase "Manage Edge Devices\n(US-01, 02, 06, 07)" as UC_ManageEdge
  usecase "Configure Detections\n(US-14, 15, 16)" as UC_ConfigDetect
  usecase "Approve/Monitor Missions\n(US-08, 09, 11, 12)" as UC_ApproveMission
  usecase "Manage Models & Jobs\n(US-20, 22, 23)" as UC_ManageModels
  usecase "Review Media & Events\n(US-17, 18, 19)" as UC_ReviewMedia
  usecase "Manage System Config\n(US-26, 27, 29)" as UC_ManageConfig
  
  usecase "Trigger Intrusion Response\n(FR-A, FR-B, US-36)" as UC_Trigger
  usecase "Execute Drone Mission\n(FR-C, FR-D, US-10)" as UC_Execute
  usecase "Transfer Data to OSC\n(FR-E, US-37)" as UC_Transfer
  usecase "Orchestrate Tapis Job\n(FR-F, US-22)" as UC_Job
}

actor "Parrot Anafi Drone" as ext_Drone
actor "Tapis API" as ext_Tapis
actor "OSC Storage" as ext_OSC

fieldTech -- UC_ManageEdge

scientist -- UC_ConfigDetect
scientist -- UC_ReviewMedia

droneOp -- UC_ApproveMission

mlEng -- UC_ManageModels
mlEng -- UC_ReviewMedia

admin -- UC_ManageConfig
admin -- UC_ManageModels

UC_Trigger .> UC_Execute : <<include>>
UC_Execute .> UC_Transfer : <<include>>
UC_Transfer .> UC_Job : <<include>>

UC_Execute -- ext_Drone
UC_Transfer -- ext_OSC
UC_Job -- ext_Tapis

@enduml