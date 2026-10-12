# <img src="assets/icons/graph.svg" width="28" height="28" alt=""> Local MLflow for the Hands-On

DSAgt records its traces in [MLflow](https://mlflow.org): one trace per agent turn and one per code run. Mission 5 reads them with `dsagt traces` and `dsagt info`. This page shows how to install MLflow, check it, and, if you want one viewer for several projects, run your own local tracking server. For the full reference, see the DSAgt [observability documentation](https://github.com/AI-ModCon/dsagt/blob/main/docs/observability.md).

## <img src="assets/icons/checklist.svg" width="22" height="22" alt=""> Install

MLflow comes with DSAgt (`mlflow>=3.11,<4.0`); there is nothing else to install. Check it from the Python environment that has dsagt:

```bash
mlflow --version
```

> [!NOTE]
> **Expect:** `mlflow, version 3.11` or later. If the command is not found, activate the environment where you ran `pip install "git+https://github.com/AI-ModCon/dsagt.git"`.

## <img src="assets/icons/gear.svg" width="22" height="22" alt=""> Default: a File in the Project

With no configuration, DSAgt writes traces to a SQLite file in the project, `~/dsagt-projects/battery-life/mlflow.db`. No server runs during the session. To open the viewer:

```bash
dsagt traces battery-life
```

It first collects the last turn's trace, then starts the MLflow viewer on http://127.0.0.1:5000 and prints a link to the project's Traces tab. Press Ctrl-C to stop it.

On macOS, port 5000 is often taken by the AirPlay Receiver. Pick another port:

```bash
dsagt traces battery-life --port 5001
```

The same file opens without dsagt, though without collecting the last turn first:

```bash
mlflow ui --backend-store-uri sqlite:///$HOME/dsagt-projects/battery-life/mlflow.db --port 5001
```

## <img src="assets/icons/plug.svg" width="22" height="22" alt=""> Optional: Your Own Local Tracking Server

Run a server when you want every DSAgt project in one viewer, or a viewer that stays open across sessions.

**1. Start the server** in a terminal of its own, and leave it running:

```bash
mkdir -p ~/mlflow
mlflow server --backend-store-uri sqlite:///$HOME/mlflow/mlflow.db --host 127.0.0.1 --port 5001
```

> [!NOTE]
> **Expect:** `curl http://127.0.0.1:5001/health` prints `OK`, and http://127.0.0.1:5001 opens the MLflow UI.

**2. Point DSAgt at it before `dsagt init`.** The URI is written into the agent's configuration at init, so export it first:

```bash
export MLFLOW_TRACKING_URI=http://127.0.0.1:5001
dsagt init
```

Add the `export` to your shell profile to keep it for later terminals.

**3. Work as usual.** The CLI, the agent's DSAgt server, and every recorded code run log to the server. `dsagt traces battery-life` prints a link into it instead of starting a viewer, and `dsagt info battery-life` reads from it.

Each project logs to its own experiment, named `dsagt-` followed by eight hex characters; the project's name is in the experiment's description. To choose the name yourself, set `mlflow.experiment` in the project's `.dsagt/config.yaml`.

> [!IMPORTANT]
> **Start the server before `dsagt start`, and keep it running until the session ends.** DSAgt cannot log to a server that is down. To switch a project back to its own file, or to another server, change `MLFLOW_TRACKING_URI` (or `unset` it) and run `dsagt init` again: exporting a new value alone moves the CLI but not the agent's DSAgt server.

## <img src="assets/icons/alert.svg" width="22" height="22" alt=""> Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `Address already in use`, or the page at port 5000 is not MLflow | Another process holds the port (AirPlay on macOS) | Use `--port 5001` |
| `No trace store yet for 'battery-life'` | No session has run in the project | Run `dsagt start battery-life` first |
| The last question you asked is missing | That turn is collected at the next session start | Open the viewer with `dsagt traces`, which collects it first |
| The UI opens on an empty Runs list | DSAgt writes traces, not runs | Open the **Traces** tab, or use the link `dsagt traces` prints |
| Traces go to the server when you expected the project file | `MLFLOW_TRACKING_URI` is still exported | `unset MLFLOW_TRACKING_URI`, then `dsagt init` again |

## <img src="assets/icons/trash.svg" width="22" height="22" alt=""> Cleanup

`dsagt rm battery-life -y` removes the project and its `mlflow.db`. A local server's store is separate: stop the server with Ctrl-C and delete `~/mlflow/` when you no longer need it.
