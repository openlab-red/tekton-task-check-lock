# Tekton Task: check-lock

Checks for concurrent PipelineRuns on the same repository. If another run is already in progress, logs a message and exits gracefully. Prevents race conditions in release pipelines.

## Usage

### With Git Resolver

```yaml
taskRef:
  resolver: git
  params:
    - name: url
      value: https://github.com/openlab-red/tekton-task-check-lock.git
    - name: revision
      value: main
    - name: pathInRepo
      value: task/check-lock.yaml
params:
  - name: repo-url
    value: $(params.repo-url)
  - name: pipeline-name
    value: "release-pipeline"
```

## Parameters

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `repo-url` | string | required | Repository URL to match against other PipelineRuns |
| `pipeline-name` | string | required | Pipeline name to filter PipelineRuns |
| `fail-on-conflict` | string | `false` | If `true`, fail with error. If `false`, exit 0 (skip gracefully) |
| `image` | string | `quay.io/openlab-red/ci-tools:latest` | Image with oc/kubectl and jq |

## Behavior

| Scenario | Behavior |
|----------|----------|
| No concurrent runs | Logs "No concurrent runs detected", proceeds |
| Concurrent run detected | Logs "Another run is already in progress", exits 0 (skip) |
| Concurrent run + `fail-on-conflict=true` | Logs message, exits 1 (fail) |

## Prerequisites

### RBAC

The pipeline ServiceAccount needs permission to list PipelineRuns. The default `pipeline` ServiceAccount in OpenShift Pipelines already has this permission.

## Example Pipeline Integration

```yaml
apiVersion: tekton.dev/v1
kind: Pipeline
spec:
  params:
    - name: repo-url
      type: string

  tasks:
    - name: check-lock
      taskRef:
        resolver: git
        params:
          - name: url
            value: https://github.com/openlab-red/tekton-task-check-lock.git
          - name: revision
            value: main
          - name: pathInRepo
            value: task/check-lock.yaml
      params:
        - name: repo-url
          value: $(params.repo-url)
        - name: pipeline-name
          value: release-pipeline

    - name: clone
      runAfter:
        - check-lock
      # ... rest of pipeline
```

## How It Works

1. Task receives `repo-url` and `pipeline-name` params
2. Queries Kubernetes API for PipelineRuns with matching pipeline label
3. Filters for runs with `status.conditions[0].reason == "Running"`
4. Compares `repo-url` param of each PipelineRun
5. Excludes the current PipelineRun from the list
6. If another running PipelineRun with same repo-url is found:
   - Logs the name of the conflicting run
   - Exits 0 (graceful skip) or 1 (fail) based on `fail-on-conflict`
7. If no conflicts, proceeds normally
