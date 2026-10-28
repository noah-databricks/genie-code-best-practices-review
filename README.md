# Databricks best practices review: a Genie Code skill

A [Genie Code](https://docs.databricks.com/aws/en/genie-code) skill that reviews a Databricks project against official Databricks best practices. Point it at a folder of notebooks, pipeline code, job or bundle config, ML/AI code or an app, and it returns a prioritised list of Must-fix, Should-fix and Optional findings. Each finding includes a fix and a link to the relevant docs page.

The skill is read-only. It reads files and runs read-only queries, and it doesn't change your code or objects.

## What it checks

- Unity Catalog and governance (three-level names, volumes vs DBFS, grants, comments)
- Security (secrets, service principals)
- Delta tables (liquid clustering, predictive optimization)
- Lakeflow Spark Declarative Pipelines (current API, AUTO CDC, expectations)
- Jobs and orchestration (idempotency, retries, job vs all-purpose compute)
- Compute and cost (serverless, warehouse sizing, auto-stop)
- Deployment and CI/CD (Declarative Automation Bundles, Git, parameterisation, tests)
- ML and AI (MLflow, Unity Catalog models, agent tracing and evaluation)
- Databricks Apps (auth, resources)
- Portability (hard-coded values that block reuse in another workspace)

## Set up (about 2 minutes)

### 1. Add this repo as a Git folder

1. In your Databricks workspace, click **Workspace** in the sidebar and open your user folder.
2. Click **Create > Git folder**.
3. Paste this repo's URL into **Git repository URL**, choose **GitHub** as the provider, and click **Create Git folder**.

This is a public repo, so you don't need Git credentials to clone it. See [Configure Git integration for Git folders](https://docs.databricks.com/aws/en/repos/repos-setup).

### 2. Register the folder with Genie Code

1. Open the Genie Code pane and click **Settings**.
2. In the row for adding a skill, enter the path to the Git folder you just created, for example `/Users/<you@company.com>/genie-code-best-practices-review`, or use the browse button to select it.
3. Click **Add folder**. `databricks-best-practices-review` appears in your skills list. Make sure its toggle is on.

See [Add skills from another folder](https://docs.databricks.com/aws/en/genie-code/skills).

To make the skill available to everyone in the workspace, a workspace admin can copy the `databricks-best-practices-review` folder into `Workspace/.assistant/skills/` instead.

## Use it

Start a new Genie Code chat and ask it to review a project, for example:

```
@databricks-best-practices-review Review the project in /Workspace/Users/<you>/<project-folder>
```

You can also open a notebook or file in the project and ask "Review this project against Databricks best practices". Genie Code loads the skill when the request matches.

The review returns:

- A summary and a status for each area
- Must-fix, Should-fix and Optional findings, each with the file or object, why it matters, the fix, and a docs link
- The top 5 fixes to make before the project goes to production or a client

## Get updates

In the Git folder, click the branch name and then **Pull** to get the latest version. Start a new Genie Code chat afterwards so the updated skill loads.
