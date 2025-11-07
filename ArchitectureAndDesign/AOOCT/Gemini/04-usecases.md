# Use Case Diagram

```mermaid
flowchart LR
  VR[Vision Researcher]
  RC[Retinal Clinician]
  IT[Imaging Technician]
  LB[Labeler / Adjudicator]
  MLE[ML Engineer]
  ADMN[Admin]
  QA_ACT[QA]
  AUD_ACT[Auditor]
  PI_ACT[PI]
  DM[Data Manager]
  OP[Operator]

  subgraph AO-OCT-MAS
    UC_Upload[UC1: Upload case & start run\n(S01)]
    UC_QC[UC2: Auto motion QC\n(S02)]
    UC_Cones[UC3: View cone maps + uncertainty\n(S03)]
    UC_CapStats[UC4: Capillary stats export\n(S04)]
    UC_EditMask[UC5: Edit masks with versions\n(S05)]
    UC_AL[UC6: Active-learning tiles\n(S06)]
    UC_ModelReg[UC7: Register model card\n(S07)]
    UC_Canary[UC8: Canary deploy & rollback\n(S08)]
    UC_SSO_RBAC[UC9: SSO + RBAC access\n(S09)]
    UC_Prov[UC10: View provenance manifest\n(S10)]
    UC_Overlay[UC11: Overlay visualization\n(S11)]
    UC_DeID[UC12: De-identified exports\n(S12)]
    UC_Drift[UC13: Drift alerts\n(S13)]
    UC_Cohort[UC14: Scheduled cohort export\n(S14)]
    UC_Golden[UC15: Golden-case regression suite\n(S15)]
    UC_Adj[UC16: Multi-rater adjudication\n(S16)]
    UC_Retry[UC17: Idempotent retries\n(S17)]
    UC_Audit[UC18: Access log reports\n(S18)]
  end

  VR --> UC_Upload
  VR --> UC_Prov
  VR --> UC_Cones
  VR --> UC_CapStats
  VR --> UC_Cohort

  RC --> UC_Cones
  RC --> UC_Overlay

  IT --> UC_Upload
  IT --> UC_QC
  IT --> UC_Retry

  LB --> UC_EditMask
  LB --> UC_AL
  LB --> UC_Adj
  LB --> UC_Overlay

  MLE --> UC_ModelReg
  MLE --> UC_Canary
  MLE --> UC_Drift
  MLE --> UC_Golden

  ADMN --> UC_SSO_RBAC

  QA_ACT --> UC_Golden

  AUD_ACT --> UC_Audit

  PI_ACT --> UC_Cohort

  DM --> UC_DeID

  OP --> UC_Retry

  %% SSO/RBAC implicitly used by all
  UC_Upload --- UC_SSO_RBAC
  UC_EditMask --- UC_SSO_RBAC
  UC_Audit --- UC_SSO_RBAC
```

*Caption: Use cases mapped directly to S01–S18 with shared SSO/RBAC.*
