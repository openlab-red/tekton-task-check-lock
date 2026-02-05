# Check Lock Task

Tekton Task for checking if a lock exists to prevent concurrent pipeline runs.

## Usage

```yaml
taskRef:
  resolver: git
  params:
    - name: url
      value: https://azuredevops.alinma.internal/DevSecOps/tekton-tasks/_git/tekton-task-check-lock
    - name: revision
      value: main
    - name: pathInRepo
      value: task/check-lock.yaml
```

## Parameters

| Name | Default | Description |
|------|---------|-------------|
| `lock-name` | required | Name of the lock to check |
| `namespace` | current | Namespace to check |

## Results

| Name | Description |
|------|-------------|
| `locked` | Whether lock exists (true/false) |

## Related Documentation

- [pipeline-release](/docs/default/component/pipeline-release/) - Release pipeline
