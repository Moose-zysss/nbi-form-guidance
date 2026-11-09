# NBI Clearance Form Guidance

Our Chapter 6 prototype will provide shared labels, examples, and help text for a sample NBI Clearance form. It follows our Chapter 5 recommendation to keep guidance in one shared form definition.

## Project status

Project setup only. The API and tests have not been implemented. The contract is a draft awaiting instructor approval.

## Tools

- Python
- FastAPI for the API
- pytest for tests
- An in-memory repository for sample data

## Folder guide

- `app/controllers/`: receive requests and return responses
- `app/services/`: apply the component's rules
- `app/repositories/`: repository interface and mock storage
- `app/models/`: field definitions and request/response models
- `tests/`: service unit tests and API checks
- `docs/`: contract, diagram, and defect log

## Prepare your environment

Run these commands from the project folder in PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
```

Dependencies are initial package choices. Record tested versions when implementation begins.

## Run and test

Run instructions will be added once the API exists. After tests are written, the test command will be:

```powershell
.\.venv\Scripts\python.exe -m pytest
```

There are no tests yet, so running pytest now will not show a passing test suite.

## Team

| Member | Role |
|---|---|
| John Moises A. Abadines | To be assigned |
| Frenz Jay D. Cabaluna | To be assigned |
| Aleah Kaira Mohammad | To be assigned |
| Chrisma J. Sigal | To be assigned |
| Jaybe Kyle S. Tacal | To be assigned |

## Team workflow

Create a GitHub or GitLab repository and protect `main`. Work on feature branches and have another member review each pull request. Each member needs at least five meaningful commits. Do not share commit identities or create empty commits to meet the count.

Have the instructor approve `docs/contract.md` before implementation. Tag the approved commit `contract-v1`. The final submission tag will be `v1.0-ch6`.

## Acknowledgment

We used ChatGPT/Codex to help prepare the contract and project setup. Our group will review and explain the submitted work.
