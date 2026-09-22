# 🎯 First: Understand What They Mean

From the JD:

- Automate compilation of PL/SQL packages, procedures, functions
- Automate data warehouse processes
- Deploy database changes via CI/CD

So they care about:

- 👉 Version control
- 👉 Safe deployment
- 👉 Automation
- 👉 Rollback strategies
- 👉 Environment control

Not deep SQL syntax.


# 1️⃣ What is PL/SQL? (Simple Explanation)

PL/SQL is Oracle’s procedural SQL language. Think of it like: Stored code that runs inside the database.

Similar to:

- Stored procedures in SQL Server
- Functions in PostgreSQL

Example:

```
CREATE OR REPLACE PROCEDURE update_salary IS
BEGIN
  UPDATE employees SET salary = salary * 1.1;
END;
/
```

# 2️⃣ What Is Data Warehouse Deployment?

In data warehouse systems, you deploy:

- Tables
- Views
- Indexes
- Stored procedures
- ETL logic
- Scheduled jobs

Traditionally (legacy way): DBA manually runs SQL scripts.

Modern DevOps way: That’s what they want you to lead.

- Scripts stored in Git
- Version controlled
- Deployed via CI/CD

**Note:** Database changes must be versioned, repeatable, and environment-aware.

# 3️⃣ How to Automate PL/SQL Deployment (Interview Answer)

I would version control all PL/SQL scripts in Git, organize them in ordered migration folders, and design a GitLab pipeline that **validates, compiles, and deploys** them to the **target environment using SQL*Plus** or a migration tool like Liquibase. Environment credentials would be stored as protected variables, and deployments would follow a controlled promotion flow.


# 4️⃣ How Would You Structure DB Repo?

Example structure:

```
database/
  migrations/
    V1__create_tables.sql
    V2__create_procedures.sql
    V3__add_indexes.sql
  rollback/
    R1__rollback_tables.sql
```

Version-based approach:

- V1
- V2
- V3

Never modify old migrations. Always add new ones. That ensures consistency.

# 5️⃣ How to Deploy via GitLab

Pipeline job example:

```
deploy_db:
  stage: deploy
  script:
    - sqlplus $DB_USER/$DB_PASS@$DB_HOST @database/migrations/V3__add_indexes.sql
```

Credentials stored in GitLab variables:

- Masked
- Protected
- Environment-specific


# 🔥 VERY IMPORTANT: Idempotency

What if script runs twice?

- Use CREATE OR REPLACE
- Upon successful migration, write to a dedicated table the status and use conditional checks against it.
- Use conditional checks and do it only when changes are made under a folder (Add the version in the Variables under CI/CD and conditionally check against it in the pipeline and replace it each time with a newly deployed version once successful)
- Use migration tool to track applied versions


# 6️⃣ Rollback Strategy (Critical Question)

For rollback, I would maintain **backups**, **versioned rollback scripts** or **rely on migration tooling** that tracks schema versions. In production, I ensure deployment first runs in staging, and **backups or snapshots are taken before applying destructive changes**.

```
variables:
  AUTOPIPELINE:
    value: "false"
    description: "set to true to run auto-pipeline-system-test in systemtest project"
    options:
      - "false"
      - "true"
  UPGRADE:
    value: "false"
    description: "set to true to run auto-upgrade-test with auto-pipeline-system-test in systemtest project"
    options:
      - "false"
      - "true"
```

# 7️⃣ Data Warehouse Specific Thinking

Data warehouse deployments often affect:

- ETL jobs
- Batch jobs
- Scheduled processes
- Large datasets

So you should say:

I ensure **database deployments are coordinated with ETL schedules to avoid data corruption** or partial transformations. That shows real thinking.


# 🔥 Likely Interview Questions

## 1️⃣ How do you version control database changes?

Answer: Migration scripts, tagged releases, Git history.

## 2️⃣ How do you prevent production data corruption?

Answer: Staging validation, transaction control, backups, approvals.

## 3️⃣ How do you test database changes?

Answer:

- Deploy to dev
- Run automated validation queries
- Smoke tests
- Verify object compilation

## 4️⃣ How do you handle environment differences?

Answer:

- Environment-specific variables
- Separate DB credentials
- Separate schemas
- Controlled promotion

## Prepare short answers for:

- How to version DB changes → Git history + migration scripts + tagged releases
- How to deploy → CI pipeline + SQL tool
- How to rollback → backups + versioned rollback scripts + migration tooling
- How to handle environments → Env specific variables + Separate DB creds + controlled promotion
- How to modernize legacy → automate manual DBA steps