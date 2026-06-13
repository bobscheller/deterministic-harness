##### Model to Harness Interface

The Model to Harness interface is the direct connection between the agent and the production floor. It is a critical part of the system as it defines the boundaries between the agent and the physical world. Therefore, the system requires the construction of the interface in addition to the actual rules engine code. The interface defines how the model behaves with the harness.

*Status: proposed design.* This interface contract is part of an architectural proposal that has not been implemented or deployed. The schemas, the worked decision example (including the APPROVED_WITH_MODIFICATION case), and all example payloads and field values are illustrative; any timeout, latency, throughput, or rule-count figures are design targets, not measured results. See the "Status and Validation" section of the parent Technical Brief for the canonical project status.

[Parent Technical Brief](technical_architecture_brief.md)

**Model to Harness Interface Specification**
```json
// ============================================================
// LAYER 2: AGENT PROMPT CONTRACT
// The agent constructs this prompt payload and sends it to
// the SLM running inside the sovereign shard.
// ============================================================

{
  "request_id": "req-a1b2c3d4-5678-90ef-ghij-klmnopqrstuv",
  "timestamp": "2026-05-06T14:32:01.882Z",
  "shard_id": "eu-west-manufacturing-07",

  // === AGENT IDENTITY ===
  "agent": {
    "agent_id": "thermal-monitor-line3-012",
    "agent_version": "2.4.1",
    "harness_rule_version": "hr-eu-mfg-v3.7"
  },

  // === MODEL TARGET ===
  "model": {
    "model_id": "slm-thermal-anomaly-v2.4",
    "model_checksum": "sha256:a8f3e9...",
    "inference_mode": "edge_local",
    "max_tokens": 512,
    "temperature": 0.1
  },

  // === ENVIRONMENTAL CONTEXT ===
  // These are the sovereign-shard-validated telemetry values.
  // The model uses these for reasoning; the harness uses
  // these to validate the model's output makes sense.
  "context": {
    "equipment": {
      "equipment_id": "CNC-MUN-L3-012",
      "equipment_type": "cnc_mill_5axis",
      "equipment_location": "eu-west-1::plant-munich::line-3::station-12",
      "operating_state": "active_cutting",
      "hours_since_maintenance": 847,
      "maintenance_threshold_hours": 1000
    },
    "thermal": {
      "ambient_temperature_c": 87.3,
      "baseline_temperature_c": 62.0,
      "delta_from_baseline_c": 25.3,
      "trend_15min": "rising",
      "trend_slope_c_per_min": 0.42,
      "source_sensor": "thermocouple-station12-A",
      "cross_validation_sensor": "ir-camera-line3-overhead",
      "cross_validation_temperature_c": 89.1,
      "sensor_agreement": true
    },
    "production": {
      "current_job_id": "JOB-2026-05-06-1142",
      "material": "titanium_ti6al4v",
      "cycle_position_pct": 73,
      "parts_completed_today": 47,
      "defect_rate_today_pct": 0.8
    }
  },

  // === PROMPT ===
  "prompt": {
    "system": "You are a thermal anomaly assessment agent for CNC manufacturing equipment. You evaluate sensor data and recommend actions. You MUST respond in the exact JSON schema defined below. Do not include any text outside the JSON response. Your role is advisory—all actions are subject to deterministic harness validation before execution.",

    "user": "Evaluate the following thermal condition for equipment CNC-MUN-L3-012:\n\n- Ambient temperature: 87.3°C (baseline: 62.0°C, delta: +25.3°C)\n- 15-minute trend: rising at 0.42°C/min\n- Cross-validation: IR camera reads 89.1°C (sensors AGREE)\n- Equipment state: active cutting, titanium Ti-6Al-4V\n- Cycle position: 73% complete\n- Hours since maintenance: 847 of 1000 threshold\n\nAssess severity, recommend an action, and provide your reasoning. Respond ONLY in the required JSON format.",

    "response_schema": {
      "type": "object",
      "required": ["assessment", "recommended_action", "reasoning", "confidence"],
      "properties": {
        "assessment": {
          "type": "object",
          "required": ["severity", "anomaly_type", "risk_level"],
          "properties": {
            "severity": {
              "type": "string",
              "enum": ["NOMINAL", "ADVISORY", "WARNING", "CRITICAL", "EMERGENCY"]
            },
            "anomaly_type": {
              "type": "string",
              "enum": [
                "NONE",
                "THERMAL_DRIFT",
                "THERMAL_SPIKE",
                "SENSOR_DEGRADATION",
                "COOLANT_FAILURE",
                "BEARING_FRICTION",
                "TOOL_WEAR",
                "UNKNOWN"
              ]
            },
            "risk_level": {
              "type": "number",
              "minimum": 0.0,
              "maximum": 1.0,
              "description": "0.0 = no risk, 1.0 = immediate danger"
            }
          }
        },
        "recommended_action": {
          "type": "object",
          "required": ["action_code", "urgency_seconds", "parameters"],
          "properties": {
            "action_code": {
              "type": "string",
              "enum": [
                "CONTINUE",
                "REDUCE_FEED_RATE",
                "PAUSE_AFTER_CYCLE",
                "PAUSE_IMMEDIATE",
                "EMERGENCY_STOP",
                "REQUEST_INSPECTION",
                "SCHEDULE_MAINTENANCE"
              ]
            },
            "urgency_seconds": {
              "type": "integer",
              "description": "Seconds before action should execute. 0 = immediate."
            },
            "parameters": {
              "type": "object",
              "description": "Action-specific parameters",
              "properties": {
                "feed_rate_reduction_pct": { "type": "number" },
                "coolant_increase_pct": { "type": "number" },
                "notification_targets": {
                  "type": "array",
                  "items": { "type": "string" }
                }
              }
            }
          }
        },
        "reasoning": {
          "type": "string",
          "maxLength": 500,
          "description": "Brief explanation of the assessment logic"
        },
        "confidence": {
          "type": "number",
          "minimum": 0.0,
          "maximum": 1.0
        }
      }
    }
  }
}
```

```json
// ============================================================
// LAYER 2 → LAYER 3: MODEL RESPONSE
// This is what the SLM returns. It goes DIRECTLY to the
// Deterministic Harness for validation before any action
// is taken.
// ============================================================

{
  "request_id": "req-a1b2c3d4-5678-90ef-ghij-klmnopqrstuv",
  "response_id": "res-x9y8z7w6-5432-10ab-cdef-ghijklmnopqr",
  "timestamp": "2026-05-06T14:32:02.114Z",
  "model_id": "slm-thermal-anomaly-v2.4",
  "inference_latency_ms": 232,

  // === MODEL OUTPUT (matches response_schema) ===
  "output": {
    "assessment": {
      "severity": "WARNING",
      "anomaly_type": "TOOL_WEAR",
      "risk_level": 0.72
    },
    "recommended_action": {
      "action_code": "REDUCE_FEED_RATE",
      "urgency_seconds": 30,
      "parameters": {
        "feed_rate_reduction_pct": 25,
        "coolant_increase_pct": 15,
        "notification_targets": [
          "shift-supervisor-munich-L3",
          "maintenance-queue-munich"
        ]
      }
    },
    "reasoning": "Temperature delta of +25.3C with rising trend at 0.42C/min on titanium cutting is consistent with progressive tool wear rather than coolant failure, as both thermal sensors agree and the pattern is gradual rather than spike. At 847 of 1000 maintenance hours with this thermal profile, recommend feed rate reduction to slow wear progression and complete current cycle safely. Maintenance inspection should be scheduled within next shift.",
    "confidence": 0.84
  },

  // === METADATA FOR HARNESS VALIDATION ===
  "validation_metadata": {
    "schema_compliant": true,
    "all_required_fields_present": true,
    "enum_values_valid": true,
    "output_token_count": 187,
    "raw_output_hash": "sha256:7c2f1b..."
  }
}
```

```json
// ============================================================
// LAYER 3: HARNESS VALIDATION DECISION
// The Harness checks the model output against the Golden
// Rule set and either approves, modifies, or blocks.
// This record goes to the Utility Monitor.
// ============================================================

{
  "request_id": "req-a1b2c3d4-5678-90ef-ghij-klmnopqrstuv",
  "response_id": "res-x9y8z7w6-5432-10ab-cdef-ghijklmnopqr",
  "harness_decision_id": "hd-m3n4o5p6-7890-12cd-efgh-ijklmnopqrst",
  "timestamp": "2026-05-06T14:32:02.118Z",
  "harness_rule_version": "hr-eu-mfg-v3.7",

  // === VALIDATION RESULTS ===
  "rules_evaluated": [
    {
      "rule_id": "THERMAL-001",
      "rule_name": "max_operating_temperature",
      "description": "Equipment must not operate above 95C for titanium cutting",
      "result": "PASS",
      "detail": "Current 87.3C is below 95C threshold"
    },
    {
      "rule_id": "THERMAL-002",
      "rule_name": "thermal_trend_escalation",
      "description": "If trend projects threshold breach within 30min, minimum action is REDUCE_FEED_RATE",
      "result": "PASS",
      "detail": "At 0.42C/min, projected to hit 95C in ~18min. Model recommended REDUCE_FEED_RATE which meets minimum required action."
    },
    {
      "rule_id": "ACTION-005",
      "rule_name": "mid_cycle_stop_prohibition",
      "description": "Titanium cutting cycles must not be interrupted mid-cut unless EMERGENCY severity",
      "result": "PASS",
      "detail": "Model recommended feed rate reduction, not mid-cycle stop. Cycle at 73% will complete."
    },
    {
      "rule_id": "MAINT-003",
      "rule_name": "maintenance_proximity_escalation",
      "description": "When hours_since_maintenance > 80% of threshold AND anomaly detected, SCHEDULE_MAINTENANCE must be added",
      "result": "AUGMENTED",
      "detail": "Model did not include SCHEDULE_MAINTENANCE as action. Harness is adding it as secondary action per rule MAINT-003. 847/1000 = 84.7% of threshold."
    }
  ],

  // === HARNESS DECISION ===
  "decision": "APPROVED_WITH_MODIFICATION",
  "decision_enum": ["APPROVED", "APPROVED_WITH_MODIFICATION", "BLOCKED", "ESCALATED_TO_HUMAN"],

  "final_action": {
    "primary_action": {
      "action_code": "REDUCE_FEED_RATE",
      "urgency_seconds": 30,
      "parameters": {
        "feed_rate_reduction_pct": 25,
        "coolant_increase_pct": 15,
        "notification_targets": [
          "shift-supervisor-munich-L3",
          "maintenance-queue-munich"
        ]
      }
    },
    "harness_added_actions": [
      {
        "action_code": "SCHEDULE_MAINTENANCE",
        "source_rule": "MAINT-003",
        "urgency_seconds": 3600,
        "parameters": {
          "maintenance_type": "tool_inspection",
          "priority": "HIGH",
          "notification_targets": ["maintenance-queue-munich"]
        }
      }
    ]
  },

  // === AUDIT / UTILITY MONITOR PAYLOAD ===
  "monitor_payload": {
    "rules_total": 4,
    "rules_passed": 3,
    "rules_augmented": 1,
    "rules_blocked": 0,
    "model_confidence": 0.84,
    "harness_latency_ms": 4,
    "model_action_preserved": true,
    "modification_reason": "Harness added supplementary action; did not override model recommendation"
  }
}
```
Sovereign shard routing happens before the model ever sees data. The temperature and equipment location are not merely prompt variables — they are the routing keys that determine which shard, which model version, and which harness rule set apply. A Munich sensor reading never touches a US-region shard; this is the Network Layer acting as the architecture's foundation.

The response schema is embedded in the prompt contract, not just documented externally. This lets the harness validate structurally (did the model return valid JSON matching the schema?) before it validates semantically (does the recommended action comply with the safety rules?). The two-phase validation matters: a malformed response is caught cheaply, before expensive rule evaluation.

The harness does not merely approve or reject — it can augment. The APPROVED_WITH_MODIFICATION decision type captures the architecture's "partner, not cage" principle: in this example the model makes a sound primary recommendation but omits a maintenance-escalation rule, and the harness adds the missing action without overriding the model's recommendation.

The monitor payload at the end of the decision record is what feeds the Utility Score. If rules_blocked climbs relative to rules_total across thousands of decisions, that is the Utility Decay signal. If model_action_preserved drops to false frequently, the harness is overriding the model too often — indicating that either the model needs retraining or the ruleset needs review.
