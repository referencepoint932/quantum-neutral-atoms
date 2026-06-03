# Quantum Neutral Atoms

Research library for exploring neutral-atom quantum computing with
[Qadence](https://pasqal-io.github.io/qadence/), running on real hardware and
simulators via [Azure Quantum](https://azure.microsoft.com/products/quantum) and
[Pasqal](https://www.pasqal.com/).

## Overview

Neutral-atom devices encode qubits in individual atoms held by optical tweezers,
with entanglement mediated by the Rydberg interaction. This project uses Qadence
to build and simulate analog and digital-analog programs, and targets Pasqal's
hardware through the Azure Quantum backend.

## Requirements

- Python ≥ 3.12
- [uv](https://docs.astral.sh/uv/) for dependency management

## Setup

```bash
uv sync
```

This creates a virtual environment and installs Qadence and its dependencies.

## Usage

```bash
uv run python main.py
```

## Credentials

Azure Quantum and Pasqal credentials are read from the environment and are never
committed (see `.gitignore`). Configure your Azure Quantum workspace and Pasqal
access tokens locally before submitting jobs to hardware.

## License

Research use. © Dr. Boris Milanovic.
