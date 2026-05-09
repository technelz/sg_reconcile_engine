# Security Group Sync / Reconciliation Engine

This project contains a dependency-aware AWS Security Group reconciliation tool.

## Folder Layout

```text
security-group-sync/
├── sg_reconcile_engine.py
├── sg-export.json
├── sg_reports/
├── rollback/
└── README.md
```

## Purpose

The engine compares source security groups from an exported JSON file against live target security groups in a target AWS account/VPC.

It supports:

- source-to-target SG matching
- exact-name matching
- CloudFormation tag matching
- optional normalized-name fallback matching
- canonical rule diffing
- dependency graph ordering
- shell-first SG creation
- SG reference remapping
- dry-run, report-only, and apply modes
- rollback journal generation
- optional removal of extra target rules

## Requirements

```bash
python3 -m pip install boto3
```

Validate syntax:

```bash
python3 -m py_compile sg_reconcile_engine.py
```

## Export Source / Prod Security Groups

Run this from the CLI:

aws ec2 describe-security-groups --profile prod --region us-east-1 --output json > sg-ex
port.json
```

## Recommended First Run

Always start with dry-run:

```bash
python3 sg_reconcile_engine.py \
  --json-path sg-export.json \
  --target-profile dr-profile \
  --target-region us-east-2 \
  --target-vpc-id vpc-yyyyyyyy \
  --dry-run \
  --allow-legacy
```

## Report Only

```bash
python3 sg_reconcile_engine.py \
  --json-path sg-export.json \
  --target-profile dr-profile \
  --target-region us-east-2 \
  --target-vpc-id vpc-yyyyyyyy \
  --report-only \
  --allow-legacy
```

## Apply Changes

Do this only after reviewing the dry-run report:

```bash
python3 sg_reconcile_engine.py \
  --json-path sg-export.json \
  --target-profile dr-profile \
  --target-region us-east-2 \
  --target-vpc-id vpc-yyyyyyyy \
  --yes \
  --allow-legacy
```

## Optional Rule Cleanup

Do not use this on the first run. Use only after validating matching and drift output:

```bash
--revoke-extra-rules
```

## Important Notes

- The `sg-export.json` included here is only a placeholder. Replace it with your real exported source/prod SG JSON.
- The tool skips the default security group.
- Apply mode blocks ambiguous normalized-name matches.
- Apply mode blocks unresolved SG reference dependencies unless `--allow-unresolved-dependencies` is used.
- Rollback journals are written under `rollback/` when apply mode performs reversible actions.
- Reports are written under `sg_reports/`.

## Current Limitations

- This is still a stabilization-stage reconciliation engine.
- Use dry-run first.
- Do not enable `--revoke-extra-rules` until matching and drift reports look correct.
- Advanced deletion safety is not implemented.
- Full convergence loop behavior is not implemented yet.
