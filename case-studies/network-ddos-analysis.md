# Network DDoS Analysis

## Problem

Build an operator-facing analysis system for high-volume L3/L4 traffic where a detection score alone is not enough.

The system needs to distinguish meaningful attacks from unusual-but-legitimate traffic, keep protocol context, group related targets, and let human review become part of the evidence lifecycle.

## Design principles

- Protocol-aware analysis for TCP, UDP and ICMP.
- Minimal data retrieval from Elasticsearch rather than downloading entire documents.
- Target grouping without losing per-IP visibility.
- Separation between machine detection and operator-confirmed verdicts.
- Live views that remain useful under sustained traffic.
- Read-only interaction with existing security data/rules unless a deliberate action path is used.

## Analysis flow

```text
Traffic telemetry
    ↓
Minimal field retrieval
    ↓
Protocol-specific features
    ↓
Target / range grouping
    ↓
Detection & confidence
    ↓
Operator review
    ↓
Confirmed verdict / evidence
```

## Why protocol context matters

A single feature rarely proves an attack. TCP behavior, for example, needs context around connection completion and peer behavior. ICMP and UDP require different indicators and baselines.

The design therefore avoids a one-rule-fits-all classifier.

## Human review

Machine output is deliberately separated from confirmed truth.

Operator review can classify findings as attack, normal, or unknown. Downstream intelligence should prefer confirmed evidence over raw model/detector output.

## Range-aware presentation

Large events often affect multiple addresses in the same network range. Grouping improves operator visibility, but the UI preserves protocol separation and drill-down so grouping does not hide the underlying targets.

## Performance

The UI and backend are designed around bounded result sets, pagination, filtering, selective fields and continuous refresh rather than repeatedly loading full raw logs.

## Confidentiality

No private network ranges, production thresholds, internal rule files, customer identifiers, or deployment details are included here.
