<div align="center">
<h3 align="center">TPG CLI</h3>

  <p align="center">
    A CLI to manage experiments for evolving, plotting, and replaying policies using TPG.
  </p>
</div>

## Table of Contents

<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#key-features">Key Features</a></li>
      </ul>
    </li>
    <li><a href="#built-with">Built With</a></li>
    <li><a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a>
      <ul>
        <li><a href="#evolve-a-policy">Evolve a Policy</a></li>
        <li><a href="#plot-results-from-an-environment">Plot Results from an Environment</a></li>
        <li><a href="#replay-the-best-policy">Replay the Best Policy</a></li>
        <li><a href="#cleanup-the-experiment-directory">Cleanup the Experiment Directory</a></li>
        <li><a href="#kill-the-evolution-of-an-experiment">Kill the Evolution of an Experiment</a></li>
        <li><a href="#enter-debug-mode">Enter Debug Mode</a></li>
      </ul>
    </li>
    <li><a href="#troubleshooting">Troubleshooting</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>

## About The Project

This project provides a Python command-line interface (CLI) powered by
[Click](https://click.palletsprojects.com/) to manage experiments for evolving,
plotting, and replaying policies. The tool leverages MPI through `mpirun` to execute
the underlying experiments defined in the TPG framework.

The CLI expects that you have the TPG binaries built (specifically,
the `TPGExperimentMPI` executable) and that you provide a configuration context
(via `ctx.obj`) containing appropriate values for:

- **`hyper_parameters`:** A dictionary mapping supported environment names to their
  respective parameter files.
- **`tpg`:** The $TPG environment variable that is the root of the TPG codebase

### Key Features

- **Evolve a Policy:** Launch an MPI run to evolve a policy for a specified environment.
- **Replay a Policy:** Replay the best performing policy based on experiment outputs.
- **Plot Results:** Plot statistics about experiments.
- **Clean Experiment Directory:** Remove the directory associated with the specified environment.
- **Kill Experiment Processes:** Terminate processes running the specified environment.
- **Debug Mode:** Enter debug mode with an OpenGL GDB window.

## Built With

- [Click](https://click.palletsprojects.com/)
- [Pandas](https://pandas.pydata.org/)
- Python 3

## Getting Started

### Prerequisites

- `uv` package manager

### Installation

To setup the dev environmnet

```sh
pip install uv
uv sync
```

## Usage

Here are the main commands.

### Evolve a Policy

```bash
tpg evolve <env>
```

This command will:

- Verify that the given `<env>` is supported (as defined in your `hyper_parameters`).
- Build a command to launch the TPG experiment using `mpirun` with the specified number of processes.
- Redirect logs about the experiment to the `$TPG/experiments/<env>/logs` directory. There are subdirectories for each stage of the evolutionary training process: `replacement`, `removal`, `selection`, and `timing`.

**Arguments:**

- `env` (str): The target environment

**Options:**

- `-s, --seed` (int): Specifying a specific random number to run (default: 42).
- `-p, --processes` (int): Specifying the number of parallel processes to run, this value defaults to 4.
- `-n, --num-experiments` (int): Specifying the number of experiments to executes (default: 1).

Here are the environments currently supported:

- `classic_control`
- `half_cheetah`
- `hopper`
- `humanoid_standup`
- `inverted_pendulum`
- `inverted_double_pendulum`
- `multitask`
- `multitask_half_cheetah`

**Note:** The environments supported are related to the yaml files present within `$TPG/src/configs`. The naming convention is extracting the string after `MuJoco_` and making everything lowercase and snakecase.

**Example:**
Running 3 experiments of the `inverted_pendulum` environment with 3 different seeds.

```bash
tpg evolve inverted_pendulum -n 3
```

### Plot Results from an Environment

```bash
tpg plot <env> <csv_files> <column_name>
```

This plotting command will:

- Generate a plot based on the CSV data provided from a previous evolution run.
- Utilize the specified column from the CSV for plotting.

**Arguments:**

- `env` (str): The target environment.
- `csv_files` (str): The path to the CSV files containing the results.
- `column_name` (str): The name of the column to plot.

For more information regarding arguments `csv_files` and `column_name`, visit our [Wiki](https://gitlab.cas.mcmaster.ca/kellys32/tpg/-/wikis/TPG-Generation-Plot-for-CSV-Logging-Files).

**Example:**

```bash
tpg plot half_cheetah all-timing generation_time
```

**Options:**

- `--num-x` (int): Optional number of x-axis points.
- `--num-y` (int): Optional number of y-axis points.

### Replay the Best Policy

```bash
tpg replay <env>
```

The replay command:

- Scans for a `selection._._.csv` file (generated from a previous evolve run).
- Uses helper functions to extract the best performing team's ID and checkpoint information.
- Launches an MPI run in replay mode with the given parameters. If running in a Dev Containers environment, MP4 replays would be populated within the `$TPG/experiments/<env>/videos` directory.

**Arguments:**

- `env` (str): The target environment

**Options:**

- `-s, --seed` (int): Replaying a specific seed for that environment
- `--seed-aux` (int): An auxiliary seed (default: 42) used in the replay; its exact role is determined by the underlying experiment logic.
- `-t, --task-to-replay` (int): Option for multitask experiments which task to visualize (default: 0).

**Example:**

```bash
tpg replay inverted_pendulum -s 2
```

This command would replay the best policy from a previously evolved experiment from a seed with value 2.

### Cleanup the Experiment Directory

```bash
tpg clean <env>
```

The clean command:

- Removes the directory associated with the specified environment.

**Arguments:**

- `env` (str): The target environment

**Example:**

```bash
tpg clean inverted_pendulum
```

### Kill the Evolution of an Experiment

```bash
tpg kill <env>
```

The kill command:

- Kills the processes running the specified environment, or kills all running processes if no environment is specified.

**Arguments:**

- `env` (str): The target environment (optional; will kill all experiment processes if no environment is specified)

**Example:**

```bash
tpg kill inverted_pendulum
```

### Enter Debug Mode

```bash
tpg debug <env>
```

The debug command:

- Creates an OpenGL GDB window that will allow for debugging of the environment.

**Arguments:**

- `env` (str): The target environment

**Options:**

- `-s, --seed` (int): Replaying a specific seed for that environment (default: 42)
- `--seed-aux` (int): An auxiliary seed (default: 42) used in the debug; its exact role is determined by the underlying experiment logic.

**Example:**

```bash
tpg debug inverted_pendulum
```

## Troubleshooting

If attempting to execute a `tpg` command leads you to an error such as: `Command 'tpg' not found`. Ensure your `PATH`
environment variables are up to date. You can do this by running `pipx ensurepath` and restarting your terminal.

## Acknowledgments

- This README was created using [gitreadme.dev](https://gitreadme.dev) — an AI tool that looks at your entire codebase to instantly generate high-quality README files.
