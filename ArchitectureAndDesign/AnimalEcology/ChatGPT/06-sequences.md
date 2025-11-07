# Design: Sequence Diagrams

## Seq-1 — Intrusion to Drone Mission to OSC Job

```mermaid
sequenceDiagram
title Seq-1: Intrusion → Drone Mission → OSC Job

participant EdgeCaptureScoring
participant EdgeEventPolicy
participant MissionPlannerDroneGateway
participant DroneFleet
participant EdgeDataTransfer
participant TapisOSCOrchestrator
participant TapisOSC

Note over EdgeCaptureScoring: Continuous capture & scoring (US-16, FR-001–004)

EdgeCaptureScoring->>EdgeEventPolicy: detectionEvent(eventId, score, ts)
EdgeEventPolicy->>EdgeEventPolicy: applyPolicies(AOI, species, thresholds)
EdgeEventPolicy-->>EdgeCaptureScoring: ack(eventId)

alt Event qualifies
  EdgeEventPolicy->>MissionPlannerDroneGateway: requestMission(eventId, AOI, policyVersion)
  MissionPlannerDroneGateway->>MissionPlannerDroneGateway: buildPlan(missionId, geofence)
  MissionPlannerDroneGateway-->>EdgeEventPolicy: missionProposal(missionId)

  MissionPlannerDroneGateway->>DroneFleet: uploadMission(missionId, plan, safetyChecks)
  DroneFleet-->>MissionPlannerDroneGateway: missionAccepted(missionId)
  MissionPlannerDroneGateway-->>EdgeEventPolicy: missionLaunched(missionId)

  DroneFleet-->>EdgeDataTransfer: capturedMediaNotice(missionId, mediaRefs)

  EdgeDataTransfer->>TapisOSC: uploadChunk(batchId, chunk[k], checksum, idempotencyKey=batchId)
  TapisOSC-->>EdgeDataTransfer: chunkAck(batchId, k)

  EdgeDataTransfer->>TapisOSCOrchestrator: datasetReady(batchId, manifestRef)
  TapisOSCOrchestrator->>TapisOSC: createJob(jobId, batchId, params)
  TapisOSC-->>TapisOSCOrchestrator: jobAccepted(jobId)
  TapisOSCOrchestrator-->>EdgeEventPolicy: jobStatus(jobId, state)
else Event suppressed
  EdgeEventPolicy-->>EdgeCaptureScoring: noAction()
end

par Timeouts/Errors
  EdgeDataTransfer-->>EdgeDataTransfer: onTimeout retryWithBackoff(batchId)
  MissionPlannerDroneGateway-->>EdgeEventPolicy: onDroneError missionFailed(missionId)
end

---

## Seq-2 — Resumable Data Transfer

sequenceDiagram
title Seq-2: Resumable Data Transfer with Integrity

participant EdgeDataTransfer
participant TapisOSC
participant MonitoringTelemetry

EdgeDataTransfer->>EdgeDataTransfer: scanRingBuffer()
EdgeDataTransfer->>TapisOSC: startUpload(batchId, manifest, idempotencyKey=batchId)
TapisOSC-->>EdgeDataTransfer: uploadSession(sessionId)

loop each chunk
  EdgeDataTransfer->>TapisOSC: putChunk(sessionId, idx, data, sha256)
  alt checksumOk
    TapisOSC-->>EdgeDataTransfer: chunkOk(idx)
  else checksumFail
    TapisOSC-->>EdgeDataTransfer: chunkError(idx)
    EdgeDataTransfer->>EdgeDataTransfer: retryChunk(idx, limitedRetries)
  end
end

EdgeDataTransfer->>TapisOSC: finalizeUpload(sessionId)
TapisOSC-->>EdgeDataTransfer: finalizeOk()

EdgeDataTransfer->>MonitoringTelemetry: recordTransferMetrics(batchId, bytes, duration)

alt linkFailure
  EdgeDataTransfer->>EdgeDataTransfer: persistState(sessionId, lastIdx)
  Note over EdgeDataTransfer: On restart, resume from last successful idx
end

----

# Seq-3 — Config Change Propagation

sequenceDiagram
title Seq-3: Config Change Propagation (Versioned)

participant ConfigAdminAPI
participant EdgeEventPolicy
participant EdgeCaptureScoring
participant MissionPlannerDroneGateway
participant MonitoringTelemetry

ConfigAdminAPI->>ConfigAdminAPI: updateConfig(changeSet, actor)
ConfigAdminAPI-->>MonitoringTelemetry: logConfigChange(configVersion, actor)

par Push updates
  ConfigAdminAPI-->>EdgeEventPolicy: pushConfig(configVersion, signedConfig)
  EdgeEventPolicy->>EdgeEventPolicy: validateSignature()
  EdgeEventPolicy-->>ConfigAdminAPI: ackConfig(configVersion)

  ConfigAdminAPI-->>EdgeCaptureScoring: pushModelConfig(configVersion, modelIds)
  EdgeCaptureScoring->>EdgeCaptureScoring: scheduleModelSwap()
  EdgeCaptureScoring-->>ConfigAdminAPI: ackConfig(configVersion)

  ConfigAdminAPI-->>MissionPlannerDroneGateway: pushFlightPolicy(configVersion)
  MissionPlannerDroneGateway->>MissionPlannerDroneGateway: applyPolicy()
  MissionPlannerDroneGateway-->>ConfigAdminAPI: ackConfig(configVersion)
end

alt Node missed push
  EdgeEventPolicy->>ConfigAdminAPI: getConfig(latestVersion)
  ConfigAdminAPI-->>EdgeEventPolicy: signedConfig(latestVersion)
end

MonitoringTelemetry->>MonitoringTelemetry: alertIf(configLag > threshold)

----

