---
name: databricks-best-practices-review
description: Review a Databricks project (notebooks, Python/SQL files, pipelines, jobs, bundle config, apps, ML/AI code) against official Databricks best practices and return prioritised Must-fix / Should-fix / Optional findings with docs links. Use when asked to review, audit or health-check a prototype, project folder or repo before it goes to production or to a client. Read-only.
---

# Databricks best practices review

Review code and configuration against official Databricks best practices. Produce a prioritised list of findings, each with a fix and a docs link.

## Rules

- **Read-only.** Do not edit, create, move or delete files, tables, jobs, pipelines or any other objects. Only read files and run read-only queries (`DESCRIBE`, `SHOW`, `SELECT` on `information_schema`).
- **Evidence for every finding.** Cite the file and line, cell, or object name. If you can't point to it, don't report it.
- **Report only what applies.** Skip checklist items for things the project doesn't use. Don't pad the report.
- **Link the docs.** Every finding links to one of the pages under "Reference docs".

## Process

1. **Confirm scope.** Use the folder, repo or notebooks the user named or @-mentioned. If none was given, ask which folder to review before starting.
2. **Inventory.** List the files in scope and identify which areas they touch: Unity Catalog, Delta tables, pipelines, jobs, compute, bundles/CI/CD, ML/AI, Apps, security.
3. **Check.** Run the relevant checklists below against each file. Where the code references tables, you may run read-only queries to confirm table type, clustering or comments.
4. **Classify.** Give each finding a severity:
   - **Must-fix**: security risk, data-loss risk, governance gap (e.g. bypasses Unity Catalog), or a significant cost or reliability problem.
   - **Should-fix**: suboptimal or legacy pattern, maintainability issue, moderate cost or performance impact.
   - **Optional**: newer feature available, minor optimisation, polish.
5. **Report** using the output format below.

## Checklists

### Unity Catalog and governance
- Tables referenced with three-level names (`catalog.schema.table`). Flag `hive_metastore` references.
- Files read from and written to Unity Catalog volumes. Flag DBFS paths (`dbfs:/`, `/dbfs/`) and mounts.
- Unity Catalog managed tables used unless there's a clear reason for external tables.
- Grants go to groups or service principals, not individual users. Least privilege, no `ALL PRIVILEGES` grants.
- Tables and key columns have comments. This also improves results from Genie and Genie Code.
- Sensitive columns protected with column masks or row filters where needed.

### Security
- No hard-coded tokens, passwords, keys or connection strings. Secrets come from secret scopes or Unity Catalog connections.
- No personal access tokens embedded in code. Production jobs run as a service principal.

### Delta tables
- Liquid clustering used for new tables, including streaming tables and materialized views, instead of partitioning or Z-order.
- Predictive optimization enabled for managed tables, rather than hand-scheduled `OPTIMIZE` / `VACUUM` jobs.
- No small-file write patterns (e.g. row-by-row inserts, many tiny appends in a loop).

### Lakeflow Spark Declarative Pipelines
- Python uses `from pyspark import pipelines as dp`. Flag `import dlt` as legacy.
- CDC uses AUTO CDC (`dp.create_auto_cdc_flow` / `AUTO CDC INTO`). Flag `apply_changes` as legacy.
- Streaming tables for incremental ingestion; materialized views for transformations and aggregates.
- Expectations on key tables with a deliberate action (`expect`, `expect_or_drop`, `expect_or_fail`).
- Incremental processing where possible rather than full reloads.

### Jobs and orchestration
- Tasks are idempotent and safe to retry.
- Retries configured for transient failures; timeouts and failure notifications set.
- Production work runs on serverless or job compute, not all-purpose clusters.
- Parameters used instead of hard-coded dates, paths, catalogs or environment names.

### Compute and cost
- Serverless considered first for jobs, pipelines, notebooks and SQL.
- SQL warehouses sized to the workload, with auto-stop set.
- Classic clusters (if used): autoscaling with sensible min/max, auto-termination set, current LTS runtime, no oversized nodes for small data.
- No `collect()` / `toPandas()` on large DataFrames; no unnecessary caching.

### Deployment and CI/CD
- Project packaged as a Declarative Automation Bundle (`databricks.yml`) with separate dev and prod targets.
- Code in Git; environment-specific values (catalog, schema, workspace) parameterised, not hard-coded.
- Tests exist for core transformation logic.

### ML and AI (if present)
- Experiments tracked in MLflow; models registered in Unity Catalog.
- Agents and LLM apps have tracing and evaluation with MLflow before deployment.
- Serving endpoints sized to the workload, with scale-to-zero where latency allows.

### Databricks Apps (if present)
- App authenticates with its service principal or on-behalf-of-user auth, not embedded tokens.
- Resources (warehouses, serving endpoints, Lakebase) declared as app resources, not hard-coded IDs.
- Heavy queries pushed to SQL warehouses or Lakebase, not computed in the app process.

### Portability (for reusable IP)
- Flag anything that would block reuse in another workspace or for another client: hard-coded workspace URLs, catalog names, user emails, sample data or IDs.

## When guidance is uncertain

Say so. Give 2-3 options with trade-offs, recommend the most conservative one, and suggest confirming with your Databricks account team if the decision is high-stakes.

## Output format

```markdown
## Databricks best practices review: <project name>

**Scope:** <folders/files reviewed>
**Summary:** <2-3 sentences on overall state and the biggest risks>

| Area | Status |
|------|--------|
| Unity Catalog and governance | Good / Needs attention / Gap |
| ... | ... |

### Must-fix
- **<Issue>** (`<file>:<line>` or `<object>`)
  - **Why:** <impact or risk>
  - **Fix:** <specific change>
  - **Ref:** <docs link>

### Should-fix
- (same format)

### Optional
- **<Improvement>** (`<location>`)
  - **Benefit:** <what you gain>
  - **Ref:** <docs link>

### Top 5 fixes before this goes to production or a client
1. ...

### Trade-offs noted
- <decisions where more than one approach is valid>
```

## Reference docs

- Best practices overview: https://docs.databricks.com/aws/en/getting-started/best-practices
- Well-architected lakehouse: https://docs.databricks.com/aws/en/lakehouse-architecture
- Unity Catalog: https://docs.databricks.com/aws/en/data-governance/unity-catalog/best-practices
- Volumes: https://docs.databricks.com/aws/en/volumes
- Secrets: https://docs.databricks.com/aws/en/security/secrets
- Delta Lake: https://docs.databricks.com/aws/en/delta/best-practices
- Liquid clustering: https://docs.databricks.com/aws/en/delta/clustering
- Predictive optimization: https://docs.databricks.com/aws/en/optimizations/predictive-optimization
- Lakeflow Spark Declarative Pipelines: https://docs.databricks.com/aws/en/ldp/best-practices
- AUTO CDC: https://docs.databricks.com/aws/en/ldp/cdc
- Expectations: https://docs.databricks.com/aws/en/ldp/expectations
- Jobs: https://docs.databricks.com/aws/en/cheat-sheet/jobs
- Compute: https://docs.databricks.com/aws/en/cheat-sheet/compute
- Serverless compute: https://docs.databricks.com/aws/en/compute/serverless/best-practices
- Classic compute configuration: https://docs.databricks.com/aws/en/compute/cluster-config-best-practices
- Declarative Automation Bundles: https://docs.databricks.com/aws/en/dev-tools/bundles
- CI/CD: https://docs.databricks.com/aws/en/dev-tools/ci-cd
- MLOps: https://docs.databricks.com/aws/en/machine-learning/mlops/mlops-workflow
- Models in Unity Catalog: https://docs.databricks.com/aws/en/machine-learning/manage-model-lifecycle
- Agent evaluation and monitoring: https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor
- Databricks Apps: https://docs.databricks.com/aws/en/dev-tools/databricks-apps/best-practices
