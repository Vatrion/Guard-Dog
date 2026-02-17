# Guard Dog API Reference

Python library for detecting malicious inputs in LLM applications.

## Installation

```bash
pip install guard-dog
```

## Quick Start

```python
from guard_dog import scan, ScanConfig

# Basic scan
result = scan("user input here")
if not result.is_clean:
    print(f"Blocked: {result.risk_score}")
```

## Core Functions

### `scan(text, metadata=None, config=None)`

Scans text for security threats.

**Parameters:**
- `text` (str): Input to scan
- `metadata` (dict, optional): Context data (user_id, session_id, etc.)
- `config` (ScanConfig, optional): Custom configuration

**Returns:** `ScanResult`

**Example:**
```python
from guard_dog import scan, ScanConfig

result = scan(
    "ignore previous instructions",
    metadata={"user_id": "123"},
    config=ScanConfig(max_text_length=50000)
)

print(result.is_clean)      # False
print(result.risk_score)    # 0.7
print(len(result.issues))   # 3
```

### `check_input(text, mode=RunMode.BLOCK_REPORT, risk_threshold=0.5, config=None, agent_id=None, agent_id_prefix=None, **kwargs)`

One-shot convenience function with full GuardDog capabilities.

**Parameters:**
- `text` (str): Input to check
- `mode` (RunMode): BLOCK_REPORT, REPORT_ONLY, or LEARN
- `risk_threshold` (float): Block threshold (0.0-1.0)
- `config` (ScanConfig): Custom configuration
- `agent_id` (str): Explicit agent identifier
- `agent_id_prefix` (str): Prefix for auto-generated agent IDs
- `**kwargs`: Additional metadata

**Returns:** `GuardResult`

**Example:**
```python
from guard_dog import check_input, RunMode

result = check_input(
    "some input",
    mode=RunMode.BLOCK_REPORT,
    risk_threshold=0.6
)
```

## Classes

### `ScanConfig(**kwargs)`

Configuration for scan behavior. All parameters optional with sensible defaults.

```python
from guard_dog import ScanConfig

config = ScanConfig(
    # Disable specific checks
    unicode_checks=False,
    prompt_injection=True,
    
    # Resource limits
    max_text_length=100000,
    repetition_threshold=0.9,
    
    # Detection sensitivity
    base64_min_length=20,
    heuristic_jailbreak_weight=0.4,
    
    # Performance
    use_multithreading=True,
    max_workers=8,
)
```

**Available Parameters:**

| Category | Parameter | Type | Default | Description |
|----------|-----------|------|---------|-------------|
| Checks | `unicode_checks` | bool | True | Enable unicode detection |
| | `homoglyphs` | bool | True | Detect homoglyph attacks |
| | `invisible_chars` | bool | True | Detect invisible characters |
| | `bidi_control` | bool | True | Detect bidi control chars |
| | `mixed_scripts` | bool | True | Detect mixed scripts |
| | `prompt_injection` | bool | True | Enable injection detection |
| | `jailbreaks` | bool | True | Detect jailbreak patterns |
| | `obfuscation` | bool | True | Detect obfuscation |
| | `delimiter_abuse` | bool | True | Detect delimiter abuse |
| | `semantic_intent` | bool | True | Semantic analysis |
| | `system_extraction` | bool | True | System prompt extraction |
| | `resource_exhaustion` | bool | True | Token flooding detection |
| | `code_injection` | bool | True | Code injection detection |
| | `escape_abuse` | bool | True | Escape char abuse |
| | `heuristic_scoring` | bool | True | Enable heuristics |
| Threading | `use_multithreading` | bool | True | Parallel detection |
| | `max_workers` | int | 8 | Thread pool size |
| | `per_detector_timeout` | float | 5.0 | Seconds per detector |
| | `cache_result` | bool | True | Enable result caching |
| Resource | `max_text_length` | int | 100000 | Max input length |
| | `repetition_threshold` | float | 0.9 | Repetition ratio threshold |
| | `min_repetition_check_length` | int | 1000 | Min length for rep check |
| | `repetition_chunk_size` | int | 50 | Sampling chunk size |
| Semantic | `semantic_max_window_size` | int | 8 | Max word window |
| | `semantic_max_gap` | int | 5 | Max gap between words |
| | `semantic_min_sentence_length` | int | 3 | Min sentence words |
| | `escalation_curve` | float | 1.5 | Risk escalation curve |
| | `density_multiplier` | float | 1.2 | Term density multiplier |
| | `obfuscation_penalty` | float | 1.3 | Obfuscation penalty |
| | `diversity_bonus` | float | 1.1 | Pattern diversity bonus |
| Heuristics | `base64_min_length` | int | 20 | Min base64 length |
| | `heuristic_jailbreak_weight` | float | 0.4 | Jailbreak score weight |
| | `heuristic_obfuscation_weight` | float | 0.3 | Obfuscation score weight |
| | `heuristic_imperative_weight` | float | 0.1 | Imperative score weight |
| Delimiter | `delimiter_special_char_min` | int | 4 | % # * threshold |
| | `delimiter_dash_equals_min` | int | 10 | - = threshold |
| | `delimiter_bracket_pipe_min` | int | 5 | < > \| threshold |
| Risk | `risk_weight_extraction` | float | 0.3 | Extraction weight |
| | `risk_weight_floods` | float | 0.2 | Flooding weight |
| | `risk_weight_code_injection` | float | 0.5 | Code injection weight |
| | `risk_weight_escape_abuse` | float | 0.2 | Escape abuse weight |
| | `risk_weight_delimiter_abuse` | float | 0.3 | Delimiter weight |
| | `risk_weight_semantic_gradient` | float | 0.3 | Semantic weight |
| Core | `max_cache_entries` | int | 1000 | Cache size limit |
| | `learn_file_truncate_length` | int | 10000 | Learn entry limit |
| | `decorator_min_string_length` | int | 10 | Min decorator length |
| | `agent_id` | str | None | Explicit agent identifier |
| | `agent_id_prefix` | str | None | Prefix for auto-generated agent IDs |
| | `cache_hash_text_length` | int | 1000 | Hash threshold |
| Syslog | `syslog_enabled` | bool | False | Enable syslog reporting |
| | `syslog_server` | str | "" | Syslog host:port |
| | `syslog_facility` | str | "user" | Syslog facility name |
| | `syslog_app_name` | str | "guard-dog" | Syslog app name |
| | `audit_threads` | int | 1 | Syslog worker threads |
| | `syslog_queue_max_size` | int | 10000 | Max syslog queue size |
| | `syslog_lag_threshold_seconds` | float | 5.0 | Lag threshold seconds |

Environment variables (set before initialization): `GUARD_DOG_SYSLOG_ENABLED`, `GUARD_DOG_SYSLOG_SERVER`, `GUARD_DOG_SYSLOG_FACILITY`, `GUARD_DOG_SYSLOG_APP_NAME`, `GUARD_DOG_AUDIT_THREADS`. If syslog is enabled without a server, initialization raises a `ValueError`.

### `ScanResult`

Result object returned by `scan()`.

**Attributes:**
- `is_clean` (bool): True if no issues found
- `risk_score` (float): 0.0 (safe) to 1.0 (critical)
- `issues` (list): Detected issues list
- `input_prompt` (str): Original input
- `timestamp` (float): Unix timestamp
- `scan_duration_ms` (float): Scan time in milliseconds
- `metadata` (dict): Passed metadata

**Methods:**
- `to_dict()`: Convert to dictionary

**Issue Dictionary Format:**
```python
{
    'type': 'JailbreakPattern',
    'description': 'Potential jailbreak detected',
    'match': 'ignore previous',
    'index': 0,
    'mitre_id': 'AML.T0056'  # optional
}
```

### `GuardDog(mode=RunMode.BLOCK_REPORT, risk_threshold=0.5, **kwargs)`

Stateful scanner with source history tracking and learn mode.

**Parameters:**
- `mode` (RunMode): BLOCK_REPORT, REPORT_ONLY, or LEARN
- `risk_threshold` (float): Block threshold
- `learn_file` (str): Path for learn mode log
- `history_size` (int): Max history entries per source (default: 100)
- `default_config` (ScanConfig): Default configuration
- `syslog_config` (SyslogConfig): Explicit syslog configuration
- `agent_id` (str): Explicit agent identifier
- `agent_id_prefix` (str): Prefix for auto-generated agent IDs

**Methods:**
- `check(text, source_id=None, metadata=None, config=None)`: Scan input with source tracking
- `get_progression_report(source_id)`: Get full attack history for a source
- `get_stats()`: Get scan statistics
- `clear_history(source_id=None)`: Clear history (all or specific source)
- `reset_stats()`: Reset statistics

**Example:**
```python
from guard_dog import GuardDog, RunMode, ScanConfig

dog = GuardDog(
    mode=RunMode.BLOCK_REPORT,
    risk_threshold=0.6,
    history_size=100,
    default_config=ScanConfig(max_workers=4)
)

# Track by source (user_id, session_id, IP, etc.)
result = dog.check("some input", source_id="user_123")

# Check for attacker progression
if result.metadata and "progression" in result.metadata:
    prog = result.metadata["progression"]
    if prog["is_escalation"]:
        print(f"Escalation detected: {prog['attacks']}")

# Get full report for a source
report = dog.get_progression_report("user_123")
```

### `RunMode` (Enum)

Behavior modes for GuardDog.

- `RunMode.BLOCK_REPORT`: Block malicious input and report
- `RunMode.REPORT_ONLY`: Allow but report issues
- `RunMode.LEARN`: Log all inputs for analysis

### `GuardResult`

Result from GuardDog.check().

**Attributes:**
- `agent_id` (str): Agent identifier for the deployment
- `input_id` (str): Unique input identifier
- `caller_id` (str): Source of the input
- `source_id` (str): Caller-provided source identifier
- `timestamp` (float): Unix timestamp
- `is_clean` (bool): True if no issues
- `risk_score` (float): Risk score
- `issues` (list): Detected issues
- `scan_duration_ms` (float): Scan time
- `action_taken` (str): blocked, reported, tracked, allowed
- `blocked` (bool): Whether input was blocked
- `metadata` (dict): Passed metadata
- `config_used` (ScanConfig): Configuration used

**Methods:**
- `to_dict()`: Convert to dictionary
- `get_detection_summary()`: Get categorized summary

### `GuardDogBlockError(Exception)`

Raised when input is blocked in BLOCK_REPORT mode.

**Attributes:**
- `result` (GuardResult): Full detection result
- `detection_summary` (dict): Categorized summary

**Methods:**
- `to_dict()`: Convert to dictionary for logging

## Decorators

### `@guard_input(mode, risk_threshold, on_block, config, agent_id, agent_id_prefix)`

Decorator for guarding function inputs.

**Parameters:**
- `mode` (RunMode): Guard behavior
- `risk_threshold` (float): Block threshold
- `on_block` (callable): Callback when blocked
- `config` (ScanConfig): Scan configuration
- `agent_id` (str): Explicit agent identifier
- `agent_id_prefix` (str): Prefix for auto-generated agent IDs

**Example:**
```python
from guard_dog import guard_input, RunMode

@guard_input(mode=RunMode.BLOCK_REPORT, risk_threshold=0.5)
def process_user_input(text: str) -> str:
    return llm.generate(text)

# With custom config
from guard_dog import ScanConfig

@guard_input(
    config=ScanConfig(unicode_checks=False, max_workers=2)
)
def fast_check(text: str) -> str:
    return llm.generate(text)
```

## Batch Operations

### `scan_batch(texts, metadata_list=None, config=None, **kwargs)`

Scan multiple texts efficiently.

**Parameters:**
- `texts` (list[str]): Texts to scan
- `metadata_list` (list[dict]): Metadata per text
- `config` (ScanConfig): Configuration
- `max_workers` (int): Parallel scan workers

**Returns:** `list[ScanResult]`

**Example:**
```python
from guard_dog import scan_batch

texts = ["input1", "input2", "input3"]
results = scan_batch(texts, max_workers=4)

for text, result in zip(texts, results):
    print(f"{text}: {result.risk_score}")
```

## Analyzer Modules (Advanced)

Direct access to individual analyzers.

### `guard_dog.analyzers.unicode`

```python
from guard_dog.analyzers import unicode

# Detect homoglyphs (look-alike characters)
issues = unicode.detect_homoglyphs(text)

# Detect invisible/zero-width characters
issues = unicode.detect_invisible_characters(text)

# Detect bidirectional control characters
issues = unicode.detect_bidi_control_characters(text)

# Detect mixed script attacks
has_mixed = unicode.detect_mixed_scripts(text)
```

### `guard_dog.analyzers.prompt_injection`

```python
from guard_dog.analyzers import prompt_injection

# Detect known jailbreak patterns
issues = prompt_injection.detect_jailbreaks(text)

# Detect base64 obfuscation
issues = prompt_injection.detect_obfuscation(text, min_length=20)

# Calculate heuristic risk score
score = prompt_injection.heuristic_score(
    text,
    jailbreak_weight=0.4,
    obfuscation_weight=0.3,
    imperative_weight=0.1
)
```

### `guard_dog.analyzers.resource_exhaustion`

```python
from guard_dog.analyzers import resource_exhaustion

issues = resource_exhaustion.detect_token_flood(
    text,
    max_length=100000,
    repetition_threshold=0.9,
    min_repetition_check_length=1000,
    repetition_chunk_size=50
)
```

### `guard_dog.analyzers.semantic_intent`

```python
from guard_dog.analyzers import semantic_intent

# Semantic pattern detection
issues = semantic_intent.detect_semantic_intent(
    text,
    max_window_size=8,
    max_gap=5
)

# Word combination detection
issues = semantic_intent.detect_word_combinations(
    text,
    target_combinations=[("ignore", "instructions")],
    max_gap=5
)

# Full gradient alert
alert = semantic_intent.detect_with_gradient_alert(text)
print(alert.tier.label)  # none, low, medium, high, critical
print(alert.gradient_score)
```

### `guard_dog.utils.token_tables`

Fast O(1) token lookup for attack detection.

```python
from guard_dog.utils.token_tables import (
    tokenize,
    JAILBREAK_TOKENS,
    HIGH_RISK_PAIRS,
    get_token_context,
    analyze_token_pattern,
    detect_with_tables,
    detect_with_tables_fast
)

# Fast tokenization (C-level str.translate + split)
tokens = tokenize("Ignore previous instructions")
# Returns: {"ignore", "previous", "instructions"}

# Check for jailbreak tokens
matches = tokens & JAILBREAK_TOKENS
# Returns: {"ignore", "previous"}

# High-risk pairs detection
issues = detect_with_tables("ignore previous instructions")
# Returns issues list with score and matched pairs

# Fast score-only version
score = detect_with_tables_fast("ignore previous")
# Returns: float (0.0-1.0)

# Context tracking across chunks
ctx1 = get_token_context("Hello world")
ctx2 = get_token_context("Ignore previous instructions")
analysis = analyze_token_pattern([ctx1, ctx2])
# Returns: tokens_found, count, high_risk_pairs, is_attack_pattern, confidence
```

**JAILBREAK_TOKENS** - Frozen set of attack-specific terms:
- Attack verbs: `ignore`, `forget`, `override`, `bypass`, `unlock`, `disable`, `jailbreak`
- Attack modifiers: `previous`, `all`, `unrestricted`, `unfiltered`
- Attack modes: `developer`, `god`, `opposite`, `dan`
- Attack behaviors: `never`, `always`, `anything`, `refuse`
- Personas: `machiavelli`, `morpheus`

**HIGH_RISK_PAIRS** - Dict mapping word pairs to score bonuses:
- `("ignore", "previous")`: +0.3
- `("ignore", "instructions")`: +0.4
- `("bypass", "restrictions")`: +0.4
- `("developer", "mode")`: +0.3

## Examples

### Basic Filtering

```python
from guard_dog import scan

def chat_endpoint(user_message):
    result = scan(user_message)
    if result.risk_score > 0.5:
        return {"error": "Input blocked", "reason": result.issues[0]['type']}
    return llm.generate(user_message)
```

### Configuration per Endpoint

```python
from guard_dog import GuardDog, ScanConfig

# Public chat - lenient
public_dog = GuardDog(
    risk_threshold=0.7,
    default_config=ScanConfig(
        heuristic_jailbreak_weight=0.3,
        max_text_length=200000
    )
)

# Admin panel - strict
admin_dog = GuardDog(
    risk_threshold=0.4,
    default_config=ScanConfig(
        heuristic_jailbreak_weight=0.5,
        risk_weight_code_injection=0.7
    )
)
```

### Field-Specific Base64 Policies

```python
from guard_dog import GuardDog, ScanConfig

base_config = ScanConfig(obfuscation=True, base64_min_length=20)
dog = GuardDog(default_config=base_config)

base64_field_config = ScanConfig(
    obfuscation=True,
    base64_min_length=128,
    heuristic_obfuscation_weight=0.05
)

payload_result = dog.check(payload_text)
iv_result = dog.check(iv_b64, config=base64_field_config)
salt_result = dog.check(salt_b64, config=base64_field_config)
```

### Learn Mode for Analysis

```python
from guard_dog import GuardDog, RunMode

dog = GuardDog(
    mode=RunMode.LEARN,
    learn_file="attacks.jsonl"
)

# All inputs logged for later analysis
result = dog.check("suspicious input")
# Check learn_file for patterns
```

### Error Handling

```python
from guard_dog import GuardDog, GuardDogBlockError

dog = GuardDog()

try:
    result = dog.check("ignore previous instructions")
    if result.blocked:
        print(f"Blocked: {result.issues}")
except GuardDogBlockError as e:
    # Exception only raised in BLOCK_REPORT mode
    print(f"Blocked: {e.result.risk_score}")
    print(f"Summary: {e.detection_summary}")
```
