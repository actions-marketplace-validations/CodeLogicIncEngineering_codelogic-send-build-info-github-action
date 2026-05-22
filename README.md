## About

GitHub Action to send git and build information from a workflow to a CodeLogic server using the CodeLogic send-build-info Docker image.

The repository at `scan_path` must be a Git working tree (typically after `actions/checkout`). Use `if: always()` on the step when build info should be sent even when earlier steps fail.


### Example

```yaml
name: codelogic-build-info

on:
  push:
    branches: [ "integration" ]
  pull_request:
    branches: [ "integration" ]
  workflow_dispatch:

jobs:
  codelogic-build-info:
    name: Send CodeLogic Build Info
    environment: CodeLogic Scan Env
    runs-on: ubuntu-latest
    steps:
      - name: Check out the repo
        uses: actions/checkout@v4
      - name: Send CodeLogic Build Info
        if: always()
        uses: CodeLogicIncEngineering/codelogic-send-build-info-github-action@master
        with:
          codelogic_host: ${{ vars.CODELOGIC_HOST }}
          agent_uuid: ${{ secrets.AGENT_UUID }}
          agent_password: ${{ secrets.AGENT_PASSWORD }}
          scan_path: /github/workspace
          job_name: ${{ github.workflow }}
          build_number: ${{ github.run_number }}
          build_status: ${{ job.status }}
          pipeline_system: GitHub Actions
```


### Example with build log

Capture build output to a file under the workspace, then pass it to the action:

```yaml
      - name: Build
        run: mvn -B package 2>&1 | tee "${GITHUB_WORKSPACE}/build.log"
      - name: Send CodeLogic Build Info
        if: always()
        uses: CodeLogicIncEngineering/codelogic-send-build-info-github-action@master
        with:
          codelogic_host: ${{ vars.CODELOGIC_HOST }}
          agent_uuid: ${{ secrets.AGENT_UUID }}
          agent_password: ${{ secrets.AGENT_PASSWORD }}
          scan_path: /github/workspace
          job_name: ${{ github.workflow }}
          build_number: ${{ github.run_number }}
          build_status: ${{ job.status }}
          pipeline_system: GitHub Actions
          log_file: /github/workspace/build.log
          log_lines: 50000
```


## Customizing


| Name | Type | Description |
|------|------|-------------|
| `codelogic_host` | String | The host address of the CodeLogic instance without the `/codelogic/ui/` part. |
| `agent_uuid` | String | The UUID of the Agent in CodeLogic. |
| `agent_password` | String | The password for the agent. |
| `scan_path` | String | Absolute path to the Git repository root inside the container. Defaults to `/github/workspace`. |
| `pipeline_system` | String | CI/CD system name reported to CodeLogic. Defaults to `GitHub Actions`. |
| `job_name` | String | Job or workflow name (e.g. `${{ github.workflow }}`). |
| `build_number` | String | Build number (e.g. `${{ github.run_number }}`). |
| `build_status` | String | Build status (e.g. `${{ job.status }}`). |
| `log_file` | String | Absolute path to a build log file inside the container. |
| `log_lines` | String | Trailing lines to send from the log file (min 5, max 50000). Only used when `log_file` is set. |
| `timeout` | String | Network timeout in seconds (min 5, max 300). |
| `dry_run` | boolean | Print planned work without executing. Defaults to `false`. |
| `cleanup_temp_files` | boolean | Delete temporary files after completion. Defaults to `false`. |
| `verbose` | boolean | Enable extra logging. Defaults to `false`. |


## Requirements

- Run `actions/checkout` before this action so `.git` exists under `scan_path`.
- Configure `CODELOGIC_HOST` (repository variable), `AGENT_UUID`, and `AGENT_PASSWORD` (repository secrets), or pass them via `with`.
- `scan_path` must be an absolute path. For standard GitHub-hosted runners, use `/github/workspace`.

## Troubleshooting

- **Git repository not found**: Mount the repository root at `scan_path`, not a subdirectory without `.git`.
- **Authentication errors**: Verify `codelogic_host`, `agent_uuid`, and `agent_password` match an agent created in CodeLogic (Agents > Create Agent).
- **Log file with host volume mounts**: This action cannot add extra Docker volume mounts. To use the `/log_file_path` pattern with `docker run -v`, run the image in a shell step instead (see CodeLogic CI workflow templates). For this action, write logs under `/github/workspace` and set `log_file` to that path.
- **Analyze / scan**: This image only supports `send_build_info`. Use [codelogic-java-agent-github-action](https://github.com/CodeLogicIncEngineering/codelogic-java-agent-github-action) or other CodeLogic agent images for code scans.
