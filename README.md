# kedro-databricks

[![Rye](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/rye/main/artwork/badge.json)](https://rye.astral.sh)
[![Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![codecov](https://codecov.io/gh/JenspederM/kedro-databricks/graph/badge.svg?token=0MUFV8BNRH)](https://codecov.io/gh/JenspederM/kedro-databricks)
<a href="https://codeclimate.com/github/JenspederM/kedro-databricks/maintainability"><img src="https://api.codeclimate.com/v1/badges/d5ef60eb0f20cb369b18/maintainability" /></a>
[![Python Version](https://img.shields.io/badge/python-3.11%20%7C%203.12-blue.svg)](https://pypi.org/project/kedro-databricks/)
[![PyPI Version](https://badge.fury.io/py/kedro-databricks.svg)](https://pypi.org/project/kedro-databricks/)


Kedro plugin to develop Kedro pipelines for Databricks. This plugin strives to provide the ultimate developer experience when using Kedro on Databricks. The plugin provides three main features:

1. **Initialization**: Transform your local Kedro project into a Databricks Asset Bundle project with a single command.
2. **Generation**: Generate Asset Bundle resources definition with a single command.
3. **Deployment**: Deploy your Kedro project to Databricks with a single command.

## Overview

The plugin provides a new `kedro-databricks` CLI command group with the following commands:

- `kedro databricks init`: Initialize a Kedro project for Databricks.
- `kedro databricks bundle`: Generate Asset Bundle resources definition.
- `kedro databricks deploy`: Deploy a Kedro project to Databricks.

## Prerequisites

- `Databricks CLI` must be installed and configured. See [Databricks CLI](https://docs.databricks.com/dev-tools/cli/index.html) for more information.
  - [Click here for help on installation](https://docs.databricks.com/en/dev-tools/cli/install.html)
  - [Click here for help on configuration](https://docs.databricks.com/en/dev-tools/cli/authentication.html)

## Installation

```bash
pip install kedro-databricks
```

## Usage

### Initialization

To initialize a Kedro project for Databricks, run:

```bash
kedro databricks init
```

This command will create the following files:

```
├── databricks.yml # Databricks Asset Bundle configuration
├── conf/
│   └── base/
│       └── databricks.yml # Workflow overrides
```

The `databricks.yml` file is the main configuration file for the Databricks Asset Bundle. The `conf/base/databricks.yml` file is used to override the Kedro workflow configuration for Databricks.

Override the Kedro workflow configuration for Databricks in the `conf/base/databricks.yml` file:

```yaml
# conf/base/databricks.yml

default: # will be applied to all workflows
    job_clusters:
        - job_cluster_key: default
          new_cluster:
            spark_version: 7.3.x-scala2.12
            node_type_id: Standard_DS3_v2
            num_workers: 2
            spark_env_vars:
                KEDRO_LOGGING_CONFIG: /dbfs/FileStore/<package-name>/conf/logging.yml
    tasks: # will be applied to all tasks in each workflow
        - task_key: default
          job_cluster_key: default

<workflow-name>: # will only be applied to the workflow with the specified name
    job_clusters:
        - job_cluster_key: high-concurrency
          new_cluster:
            spark_version: 7.3.x-scala2.12
            node_type_id: Standard_DS3_v2
            num_workers: 2
            spark_env_vars:
                KEDRO_LOGGING_CONFIG: /dbfs/FileStore/<package-name>/conf/logging.yml
    tasks:
        - task_key: default # will be applied to all tasks in the specified workflow
          job_cluster_key: high-concurrency
        - task_key: <my-task> # will only be applied to the specified task in the specified workflow
          job_cluster_key: high-concurrency
```

The plugin loads all configuration named according to `conf/databricks*` or `conf/databricks/*`.

### Generation

To generate Asset Bundle resources definition, run:

```bash
kedro databricks bundle
```

This command will generate the following files:

```
├── resources/
│   ├── <project>.yml # Asset Bundle resources definition corresponds to `kedro run`
│   └── <project-pipeline>.yml # Asset Bundle resources definition for each pipeline corresponds to `kedro run --pipeline <pipeline-name>`
```

The generated resources definition files are used to define the resources required to run the Kedro pipeline on Databricks.

### Deployment

To deploy a Kedro project to Databricks, run:

```bash
kedro databricks deploy
```

This command will deploy the Kedro project to Databricks. The deployment process includes the following steps:

1. Package the Kedro project for a specfic environment
2. Generate Asset Bundle resources definition for that environment
3. Upload environment-specific `/conf` files to Databricks
4. Upload `/data/raw/*` and ensure other `/data` directories are created
5. Deploy Asset Bundle to Databricks
