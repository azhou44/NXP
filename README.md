# **Databricks DataOps Repository**

[TOC]

## **Repository Structure and Branch Strategy**

This repository follows a branching strategy to manage deployments across multiple Databricks workspaces:

- **`1dp-migration-dev`** – Used for testing and initial deployments.
- **`1dp-migration-acc`** – Used for staging and validation before production.
- **`1dp-migration-prd`** – Final version, deployed to the production workspace.
- **`1dp-cons-dev`** – Consumer development workspace.
- **`1dp-cons-acc`** – Consumer acceptance workspace.
- **`1dp-cons-prd`** – Consumer production workspace.


### **Environment Variables**  
The repository has project-level environment variables to connect to each Databricks workspace. These variables are used in the pipeline to log in and deploy to the correct environment.

| Variable Name                       | Description                                       |
|------------------------------------|---------------------------------------------------|
| `databricks_sp_id_dev`             | Service principal ID for the development workspace |
| `databricks_sp_secret_dev`         | Service principal secret for the development workspace |
| `databricks_host_dev`              | The Databricks development workspace URL          |
| `databricks_sp_id_acc`             | Service principal ID for the acceptance workspace |
| `databricks_sp_secret_acc`         | Service principal secret for the acceptance workspace |
| `databricks_host_acc`              | The Databricks acceptance workspace URL           |
| `databricks_sp_id_prd`             | Service principal ID for the production workspace |
| `databricks_sp_secret_prd`         | Service principal secret for the production workspace |
| `databricks_host_prd`              | The Databricks production workspace URL           |
| `databricks_sp_id_cons_dev`        | Service principal ID for consumer dev workspace   |
| `databricks_sp_secret_cons_dev`    | Service principal secret for consumer dev workspace |
| `databricks_host_cons_dev`         | Consumer dev workspace URL                        |
| `databricks_sp_id_cons_acc`        | Service principal ID for consumer acc workspace   |
| `databricks_sp_secret_cons_acc`    | Service principal secret for consumer acc workspace |
| `databricks_host_cons_acc`         | Consumer acc workspace URL                        |
| `databricks_sp_id_cons_prd`        | Service principal ID for consumer prd workspace   |
| `databricks_sp_secret_cons_prd`    | Service principal secret for consumer prd workspace |
| `databricks_host_cons_prd`         | Consumer prd workspace URL                        |

These variables are used in the GitLab [example pipeline](#cicd-integration) to authenticate and deploy resources to the appropriate Databricks workspace based on the active branch.

## **Databricks Asset Bundles Setup and CI/CD Integration**

### **1. Prerequisites**  

Before running any commands, ensure you have the following installed:
- **Databricks CLI**
- **Python 3** and `pip`
- **Git**


### **2. Install Databricks CLI**

Run the following command to install the Databricks CLI:

```sh
curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sudo sh
```

Verify the installation:

```sh
databricks --version
```

You should see output similar to `Databricks CLI vx.xxx.x`. Ensure that you are using Databricks CLI version 0.218.0 or above, as earlier versions do not support Databricks Asset Bundles.

### **3. Authenticate with Databricks**

Authenticate with your Databricks workspace by running the following command:

```sh
databricks auth login --host <workspace-url>
```

This will open a browser window where you can sign in to your Databricks workspace.  
Follow the on-scren instructions to complete the authentication process.

### **4. Clone the Repository**

Clone this repo using your either SSH or HTTPs, then navigate into the project directory.

```sh
git clone <repository-url>
cd <repository-name>
```

Replace `<repository-url>` with the actual clone link and `<repository-name>` with the name of the repository.

### **5. Initialize the Databricks Bundle**

Run the following command to initialize a new bundle:
```sh
databricks bundle init default-python
```

After running this command, you will be prompted to:

1. **Provide a project name** – Enter a name for your Databricks bundle.
2. **Choose whether to include sample files** – You can select `yes` to include an example job and pipeline or `no` to start with a minimal setup.

This will create a new directory using the project name you provided, containing the necessary configuration files.

#### **5.1 Created Resources**

If you choose to include the sample notebook and pipeline, the following folder structure will be generated:

```
project_name/
├── README.md
├── databricks.yml
├── fixtures
├── pytest.ini
├── requirements-dev.txt
├── resources
│   ├── my_project.job.yml
│   └── my_project.pipeline.yml
├── scratch
│   ├── README.md
│   └── exploration.ipynb
├── setup.py
├── src
│   ├── dlt_pipeline.ipynb
│   ├── my_project
│   │   ├── __init__.py
│   │   └── main.py
│   └── notebook.ipynb
└── tests
    └── main_test.py
```

Files of particular interest include the following:

- **databricks.yml:** This file specifies the bundle’s programmatic name, includes a reference to the pipeline definition, and specifies settings about the target workspace.
- **resources/<project-name>_job.yml** and **resources/<project-name>_pipeline.yml**: These files define the job that contains a pipeline refresh task, and the pipeline’s settings.
- **src/dlt_pipeline.ipynb**: This file is a notebook that, when run, executes the pipeline.

### **6. Manage Your Resources**

Databricks Asset Bundles allow you to deploy various resources by adding them to the `resources/` directory.  
For details on adding additional resources, refer to the Databricks documentation on [bundle configuration examples](https://docs.databricks.com/aws/en/dev-tools/bundles/resource-examples).

## **CI/CD Integration**

The GitLab pipeline file [.gitlab-ci.yml.example](.gitlab-ci.yml.example) provides an example of how to automate validation and deployment. By default, the file is renamed to prevent the pipeline from running automatically. Rename the file to `.gitlab-ci.yml` to enable the pipeline.

```yaml
stages:
  - validate
  - deploy-dev
  - deploy-acc
  - deploy-prd
default:
  before_script:
    - |
      if [[ "$CI_COMMIT_BRANCH" == "1dp-migration-dev" ]]; then
        export DATABRICKS_CLIENT_ID=$databricks_sp_id_dev
        export DATABRICKS_CLIENT_SECRET=$databricks_sp_secret_dev
        export DATABRICKS_HOST=$databricks_host_dev
      elif [[ "$CI_COMMIT_BRANCH" == "1dp-migration-acc" ]]; then
        export DATABRICKS_CLIENT_ID=$databricks_sp_id_acc
        export DATABRICKS_CLIENT_SECRET=$databricks_sp_secret_acc
        export DATABRICKS_HOST=$databricks_host_acc
      elif [[ "$CI_COMMIT_BRANCH" == "1dp-migration-prd" ]]; then
        export DATABRICKS_CLIENT_ID=$databricks_sp_id_prd
        export DATABRICKS_CLIENT_SECRET=$databricks_sp_secret_prd
        export DATABRICKS_HOST=$databricks_host_prd
      elif [[ "$CI_COMMIT_BRANCH" == "1dp-cons-dev" ]]; then
        export DATABRICKS_CLIENT_ID=$databricks_sp_id_cons_dev
        export DATABRICKS_CLIENT_SECRET=$databricks_sp_secret_cons_dev
        export DATABRICKS_HOST=$databricks_host_cons_dev
      elif [[ "$CI_COMMIT_BRANCH" == "1dp-cons-acc" ]]; then
        export DATABRICKS_CLIENT_ID=$databricks_sp_id_cons_acc
        export DATABRICKS_CLIENT_SECRET=$databricks_sp_secret_cons_acc
        export DATABRICKS_HOST=$databricks_host_cons_acc
      elif [[ "$CI_COMMIT_BRANCH" == "1dp-cons-prd" ]]; then
        export DATABRICKS_CLIENT_ID=$databricks_sp_id_cons_prd
        export DATABRICKS_CLIENT_SECRET=$databricks_sp_secret_cons_prd
        export DATABRICKS_HOST=$databricks_host_cons_prd
      fi
    - echo "Validate the bundle"
    - cd my_project
    - databricks bundle validate
validate-job:
  stage: validate
  script:
    - databricks bundle validate
  tags:
    - databricks-dab
  only:
    - 1dp-migration-dev
    - 1dp-migration-acc
    - 1dp-migration-prd
    - 1dp-cons-dev
    - 1dp-cons-acc
    - 1dp-cons-prd
deploy-dev:
  stage: deploy-dev
  script:
    - databricks bundle deploy -t dev
  tags:
    - databricks-dab
  only:
    - 1dp-migration-dev
    - 1dp-cons-dev
deploy-acc:
  stage: deploy-acc
  script:
    - databricks bundle deploy -t acc
  tags:
    - databricks-dab
  only:
    - 1dp-migration-acc
    - 1dp-cons-acc
deploy-prd:
  stage: deploy-prd
  script:
    - databricks bundle deploy -t prd
  tags:
    - databricks-dab
  only:
    - 1dp-migration-prd
    - 1dp-cons-prd
```

**Note:** Replace `<project-name>` with the name of your project chosen in **Step 5**.

### **Extending the Pipeline**

To deploy resources to additional environments, follow the `deploy-dev` job configuration:

1. Add a new stage below `stages:` at the beginning of the file.
2. Copy and modify the `deploy-dev` job section for each environment.
3. Adjust the `-t <target>` flag in the `databricks bundle deploy` command to match the appropriate target environment.
4. Add the correct `only:` branch filters to ensure the pipeline runs for the intended Git branch.