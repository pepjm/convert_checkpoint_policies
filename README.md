# Checkpoint Firewall Policy to CSV Converter

Convert Checkpoint firewall access-layer policies from JSON to CSV with flexible field selection, object resolution, and rule splitting.

## Files

| File | Description |
|---|---|
| `checkpoint_policy.json` | Example Checkpoint R81.10 policy with objects, rules, and inline layers |
| `convert_checkpoint.py` | v1 — basic name-based resolution |
| `convert_checkpoint_v2.py` | v2 — resolves objects to IP/CIDR |
| `convert_checkpoint_v3.py` | v3 — shows Name [IP/CIDR], resolves services |
| `convert_checkpoint_v4.py` | v4 — hierarchical inline numbering, `;` separator, `--split` |
| `convert_checkpoint_v5.py` | v5 — adds `status` field (Enabled/Disabled), `policy-name`, `rule-type` |
| `convert_checkpoint_v6.py` | v6 — adds `--split-groups` flag |

## Quick start

```
python convert_checkpoint_v6.py checkpoint_policy.json "rule-number,status,name,source,destination,service,action"
```

## Available fields

```
rule-number   Policy rule number (supports hierarchical: 8.1, 8.2)
name          Rule name
rule-type     access | inline | app-control | threat-prevention
policy-name   Policy package name
status        Enabled | Disabled
enabled       Raw boolean (True/False)
source        Source objects (Name [IP/CIDR])
destination   Destination objects (Name [IP/CIDR])
service       Service objects (Name [protocol/port])
action        Accept | Drop | Prevent | Reject
track         Log | Alert | None
comments      Rule comments
content       Application/URL categories
inline-layer  Referenced inline layer name
time          Time object name
user          User object name
install-on    Installation target
threat-name   Threat name
threat-category  Threat severity categories
uid           Rule UID
_layer        Layer path (e.g. "Network Access Layer > Content Filtering - Inline Layer")
```

## Version comparison
### v6 — Group expansion (--split-groups)

```
--split-groups    Expand group objects into individual member refs
                  (composes with --split for per-member rows)
```

## Flags

| Flag | Effect |
|---|---|
| `--split` | Expand multi-source/multi-destination rules into one row per source-destination pair |
| `--split-groups` | Expand group objects into their individual member objects before resolution |

Both flags can be combined:
```
python convert_checkpoint_v6.py --split-groups --split checkpoint_policy.json "rule-number,name,source,destination,action"
```

## Examples

### Basic conversion
```
python convert_checkpoint_v6.py checkpoint_policy.json "rule-number,status,name,source,destination,action"
```

### With rule type and policy name
```
python convert_checkpoint_v6.py checkpoint_policy.json "rule-number,rule-type,policy-name,status,name,source,destination,action"
```

### Expanded view per source-destination pair
```
python convert_checkpoint_v6.py --split checkpoint_policy.json "rule-number,name,source,destination,action"
```

### Groups expanded into members
```
python convert_checkpoint_v6.py --split-groups checkpoint_policy.json "rule-number,name,source,destination"
```

### Groups expanded and fully split
```
python convert_checkpoint_v6.py --split-groups --split checkpoint_policy.json "rule-number,name,source,destination"
```

## Test data

`checkpoint_policy.json` contains:

- **15 access rules** in Network Access Layer (rules 1-15)
  - Includes stealth, anti-spoofing, DNS, web, database, SSH, outbound internet
  - Multi-source rules (rules 10, 12)
  - Multi-destination rules (rules 9, 11, 13)
  - Multi-source + multi-destination (rule 14)
  - Group as source (rule 12: All_Internal_Networks — 4 members)
  - Group as destination (rule 7: Internal_Servers — 5 members)
  - 1 disabled rule (rule 11: Legacy FTP — `enabled: false`)
  - Inline layer with 2 child rules (Content Filtering: 8.1, 8.2)
  - Unlinked inline layer (Threat Prevention: 2 rules)
- **4 application control rules** in Application & URL Filtering Layer
  - 1 disabled rule (rule 4: Legacy Chat — `enabled: false`)
- **40+ objects**: hosts, networks, groups, services (TCP/UDP/ICMP), application sites, users, time objects
