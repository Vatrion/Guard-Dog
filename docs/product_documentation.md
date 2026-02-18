# Guard Dog Product Documentation

## Overview

Guard Dog is a pure-Python security library for detecting malicious inputs in LLM applications. It focuses on ultra-low latency detection using regexes and heuristics with no runtime dependencies.

## Detection Coverage

Guard Dog scans for:

- Prompt injection and jailbreak patterns
- Unicode obfuscation (homoglyphs, invisible characters, bidirectional controls, mixed scripts)
- System prompt extraction attempts (MITRE ATLAS AML.T0056)
- Resource exhaustion and token flooding (MITRE ATLAS AML.T0029)
- Code injection patterns
- Escape character abuse
- Delimiter abuse
- Semantic intent signals and escalation patterns

## Detection Pipeline

The scanning pipeline runs in a fixed order to maximize accuracy:

1. Unicode checks on raw text
2. Prompt injection checks on normalized text
3. System extraction checks on normalized text
4. Resource exhaustion checks on raw text
5. Code injection checks on normalized text
6. Escape abuse checks on raw text
7. Delimiter abuse checks on raw text
8. Semantic intent checks on raw text

Unicode normalization removes invisible characters and normalizes homoglyphs before downstream checks.

## Runtime Modes

Guard Dog supports three run modes:

- `BLOCK_REPORT`: block malicious input and report
- `REPORT_ONLY`: allow input but report
- `LEARN`: log inputs to JSONL for analysis

## Agent Identity

Each GuardDog deployment is identified by an `agent_id` to enable evidence correlation across distributed systems.

- Set `agent_id` explicitly for a fixed identifier
- Set `agent_id_prefix` to generate `<prefix>-<counter>` IDs
- If neither is provided, Guard Dog uses `<hostname>-<counter>`

## Configuration and Customization

Configuration is controlled through `ScanConfig`. Pass a complete config instance to `scan` or `GuardDog` to override defaults.

Key customization areas:

- Enable/disable detectors (e.g., `semantic_intent=False`)
- Detection sensitivity (e.g., `repetition_threshold`, `base64_min_length`)
- Risk weighting (e.g., `risk_weight_extraction`)
- Threading controls (`use_multithreading`, `max_workers`)
- Logging controls (`syslog_enabled`, `syslog_server`)
- Evidence capture length (`learn_file_truncate_length`)

## Logging and Evidence

Guard Dog emits RFC5424 syslog messages with a JSON payload that is SIEM and GCP logging friendly. Authentication for log ingestion is handled by your environment.

Event payload highlights:

- `schema_version`, `event_time`, `severity`, `syslog_severity`
- `agent_id`, `input_id`, `caller_id`, `source_id`
- `risk_score`, `blocked`, `action_taken`, `scan_duration_ms`
- `issues`, `issue_types`, `detection_summary`
- `input.text_hash_sha256`, `input.text_sample`, `input.text_preview`
- `scan_config`, `metadata`, `runtime`, `library`
- Optional `qa_details` for debugging

Evidence controls:

- `learn_file_truncate_length` caps `input.text_sample` length
- `input.text_hash_sha256` is always included for correlation

## Learn Mode Artifacts

In `LEARN` mode, Guard Dog writes JSONL entries with:

- `event_time`, `agent_id`, `input_id`, `source_id`
- `text`, `text_length`, `text_hash_sha256`
- `issues`, `issue_types`, `risk_score`, `blocked`
- `scan_config`, `library_version`

## Integration Samples

For provider SDK and API integration patterns, see the Samples section in the Integration Guide:

- `guard_dog/docs/integration_guide.md`

## Performance Guidance

- Prefer multithreading for large inputs or high concurrency
- Disable expensive checks if not needed (e.g., `semantic_intent=False`)
- Adjust `per_detector_timeout` for tighter latency bounds

## Privacy Considerations

- Avoid placing secrets in metadata fields
- Tune `learn_file_truncate_length` to limit text captured in logs
- Use `source_id` to track attacker progression without logging user identifiers directly
