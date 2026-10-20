# Contributing to Robosapiens-AKD

Thank you for your interest in contributing to Robosapiens-AKD.

## Scope

This repository is maintained as part of the INTO-CPS ecosystem and contains the reference implementation of the AKD paper. Contributions should preserve:

- reproducibility of the CIFAR-100 results reported in the paper
- clarity of the code
- low setup complexity
- explicit documentation of dependencies and pretrained models

## How to contribute

1. Open an issue describing the proposed change.
2. Fork the repository and create a feature branch.
3. Keep changes focused and documented.
4. Submit a pull request with a short explanation.

## Contribution guidelines

- Keep `scripts/fetch_pretrained_teachers_fixed.sh` and `train_cifar_student.py` runnable end-to-end.
- Document any new dependencies in `requirements.txt` and the README.
- State the source and license of any new dataset or pretrained model.
- Do not commit datasets or model checkpoints.
- Prefer small, reviewable pull requests.

## Code style

- Follow standard Python conventions (PEP 8 where practical).
- Use descriptive names for variables and functions.

## Reporting issues

Please use GitHub Issues for bug reports, questions, and suggestions.
