# Use Case Diagram

```mermaid
actor Clinician
actor Researcher
actor ImagingTech
actor Labeler
actor MLEngineer
actor Admin
actor Auditor

usecase "Upload AO-OCT Volume" as UC1 {
    (Clinician, Researcher) --> UC1 : "Trigger pipeline run\n[Story: Researcher uploads volume]"
}
usecase "Review Metrics/Overlays" as UC2 {
    (Clinician, Researcher) --> UC2 : "Visualize results\n[Story: Clinician views density maps]"
}
usecase "Edit Masks" as UC3 {
    Labeler --> UC3 : "Submit edits\n[Story: Labeler edits masks]"
}
usecase "Adjudicate Conflicts" as UC4 {
    Labeler --> UC4 : "Resolve multi-rater conflicts\n[Story: Reviewer adjudicates]"
}
usecase "Register Model" as UC5 {
    MLEngineer --> UC5 : "Promote model version\n[Story: ML engineer registers model]"
}
usecase "Monitor Runs" as UC6 {
    ImagingTech --> UC6 : "Re-queue failed jobs\n[Story: Imaging tech monitors runs]"
}
usecase "Audit Access" as UC7 {
    Admin --> UC7 : "Export audit logs\n[Story: Admin checks compliance]"
    Auditor --> UC7
}
usecase "Schedule Exports" as UC8 {
    DataManager --> UC8 : "De-identify and export\n[Story: Data manager schedules exports]"
}

UC1 -->|includes| UC6 : "Run monitoring"
UC3 -->|extends| UC4 : "If conflict detected"
UC5 -->|includes| UC8 : "Model metrics exported"
