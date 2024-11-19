[![Geek Cell GmbH](https://raw.githubusercontent.com/geekcell/.github/main/geekcell-github-banner.png)](https://www.geekcell.io/)

[![Linter](https://github.com/geekcell/github-action-aws-ecs-run-task/actions/workflows/linter.yaml/badge.svg)](https://github.com/geekcell/github-action-aws-ecs-run-task/actions/workflows/linter.yaml)
[![Test](https://github.com/geekcell/github-action-aws-ecs-run-task/actions/workflows/test.yaml/badge.svg)](https://github.com/geekcell/github-action-aws-ecs-run-task/actions/workflows/test.yaml)
[![Check dist/](https://github.com/geekcell/github-action-aws-ecs-run-task/actions/workflows/check-dist.yaml/badge.svg)](https://github.com/geekcell/github-action-aws-ecs-run-task/actions/workflows/check-dist.yaml)

<!-- action-docs-description -->
## Description
TEST
Run an AWS ECS Fargate task and execute a custom commands. See the log output of the commands.
<!-- action-docs-description -->

### Details
This action makes it possible to run an AWS ECS Fargate task and execute custom commands. If the task definition
is configured to log to CloudWatch, this action will try to tail the output of container, providing instant feedback inside
the GitHub Workflow.

This action is great for executing migrations or other pre/post deployment steps for ECS Fargate applications.

## Usage

#### Full example
``` yaml
- name: Execute migrations and seeders
  id: run-task
  uses: geekcell/github-action-aws-ecs-run-task@v3.0.0
  with:
    cluster: application-cluster
    task-definition: application-task-def
    assign-public-ip: 'DISABLED'

    subnet-ids: |
      subnet-04f133a104b9e95df
      subnet-0dc419ee6a1483514

    security-group-ids: |
      sg-123456789101112
      sg-112398765421333

    tail-logs: true
    override-container: app
    override-container-command: |
      /bin/sh
      -c
      php artisan migrate --force --ansi && php artisan db:seed --force --ansi
    override-container-environment: |
      AWS_REGION=us-east-1
      FOO=baz

    task-wait-until-stopped: true
    task-start-max-wait-time: 120
    task-stopped-max-wait-time: 300
```

#### Minimal example
``` yaml
- name: Run migration container
  id: run-task
  uses: geekcell/github-action-aws-ecs-run-task@v3.0.0
  with:
    cluster: application-cluster
    task-definition: application-task-def
    subnet-ids: subnet-04f133a104b9e95df
    security-group-ids: sg-123456789101112
```

#### Appending multiple lines into a single command

You can use the backslash character `\` to append multiple lines into a single line. This is useful if you have many
commands to execute and want to keep the YAML file readable. Otherwise, each line will be passed to the AWS ECS Fargate
task as a separate argument.

> **Note:** Make sure to use the `|` character so the YAML parser interprets the value as a multiline string.
> You can read more about this in the [YAML documentation](https://yaml.org/spec/1.2/spec.html#id2794534).

For example:

``` yaml
...
override-container-command: |
  /bin/sh
  -c
  php artisan down && \
  php artisan migrate --force --ansi && \
  php artisan db:seed --force --ansi && \
  php artisan cache:clear --ansi
```

Will pass the following command to the container on the AWS ECS Fargate task:
```
["sh", "-c", "php artisan down && php artisan migrate --force --ansi && php artisan db:seed --force --ansi && php artisan cache:clear --ansi"]
```

<!-- action-docs-inputs -->
## Inputs

| parameter | description | required | default |
| --- | --- | --- | --- |
| task-definition | The name or the ARN of the task definition to use for the task. | `true` |  |
| subnet-ids | The list of subnet IDs for the task to use. If multiple they should be passed as multiline argument with one subnet ID per line. | `true` |  |
| security-group-ids | List of security group IDs for the task. If multiple they should be passed as multiline argument with one subnet ID per line. | `true` |  |
| assign-public-ip | Assign public a IP to the task. Options: `['ENABLED', 'DISABLED']` | `false` | DISABLED |
| cluster | Which ECS cluster to start the task in. | `false` |  |
| override-container | Will use `containerOverrides` to run a custom command on the container. If provided, `override-container-command` must also be set. | `false` |  |
| override-container-command | The command to run on the container if `override-container` is passed. | `false` |  |
| override-container-environment | Add or override existing environment variables if `override-container` is passed. Provide one per line in key=value format. | `false` |  |
| tail-logs | If set to true, will try to extract the logConfiguration for the first container in the task definition. If `override-container` is passed, it will extract the logConfiguration from that container. Tailing logs is only possible if the provided container uses the `awslogs` logDriver. | `false` | true |
| task-wait-until-stopped | Whether to wait for the task to stop before finishing the action. If set to false, the action will finish immediately after the task reaches the `RUNNING` state (fire and forget). | `false` | true |
| task-start-max-wait-time | How long to wait for the task to start (i.e. reach the `RUNNING` state) in seconds. If the task does not start within this time, the pipeline will fail. | `false` | 120 |
| task-stop-max-wait-time | How long to wait for the task to stop (i.e. reach the `STOPPED` state) in seconds. The task will not be canceled after this time, the pipeline will just be marked as failed. | `false` | 300 |
| task-check-state-delay | How long to wait between each AWS API call to check the current state of the task in seconds. This is useful to avoid running into AWS rate limits. **However**, setting this too high might cause the Action to miss the time-window your task is in the "RUNNING" state (if you task is very short lived) and can cause the action to fail. | `false` | 6 |
<!-- action-docs-inputs -->

<!-- action-docs-outputs -->
## Outputs

| parameter | description |
| --- | --- |
| task-arn | The full ARN for the task that was ran. |
| task-id | The ID for the task that was ran. |
| log-output | The log output of the task that was ran, if `tail-logs` and `task-wait-until-stopped` are set to true. |
<!-- action-docs-outputs -->

<!-- action-docs-runs -->
## Runs

This action is a `node20` action.
<!-- action-docs-runs -->
