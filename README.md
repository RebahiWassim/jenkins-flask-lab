# TP7 — Building a CI/CD Pipeline with Jenkins and Docker

Implementation of Lab No. 07 (DevOps & Cloud Computing, 5th-year Software Engineering).

## Project structure

```
jenkins-flask-lab/
├── app.py                 # Flask app: "/" and "/health" endpoints
├── requirements.txt       # Flask, pytest, gunicorn
├── Dockerfile             # python:3.12-slim image running gunicorn on port 8000
├── Jenkinsfile            # 5-stage declarative pipeline
├── tests/
│   └── test_app.py        # pytest tests for both endpoints
├── rapport_TP7_Jenkins.tex # Deliverable report (LaTeX)
└── README.md              # This file
```

## Local check (before pushing)

```bash
pip install -r requirements.txt
pytest -v
docker build -t jenkins-flask-lab:local .
docker run --rm jenkins-flask-lab:local pytest -v
docker run -d --name jenkins-flask-app -p 8081:8000 -e APP_VERSION=local jenkins-flask-lab:local
curl http://localhost:8081/health    # -> {"status":"ok"}
curl http://localhost:8081/          # -> "Version: local"
```

## Publish and configure Jenkins

1. Create an empty **public** GitHub repository named `jenkins-flask-lab`.
2. `git init && git add . && git commit -m "Add Flask application for Jenkins lab"`
3. `git branch -M main`
4. `git remote add origin https://github.com/USERNAME/jenkins-flask-lab.git`
5. `git push -u origin main`
6. In Jenkins: **New Item → Pipeline**, name `jenkins-flask-lab`.
7. Pipeline definition: **Pipeline script from SCM → Git**, repository URL above,
   branch `*/main`, script path `Jenkinsfile`. Save and click **Build Now**.
8. The Jenkins agent must be able to reach the Docker daemon (`docker info` works).
   Do not grant Docker access to Jenkins on shared/production hosts.

## Failure experiment

Change the assertion in `tests/test_app.py` to
`assert response.get_json() == {"status": "incorrect"}`, commit and push, then rebuild.
The **Test** stage fails, so **Deploy** and **Verify** are skipped and the previous
container keeps running. Restore `{"status": "ok"}`, push and rebuild to recover.

## Execution results (this run)

| Build | Result | Note |
|---|---|---|
| #1 | SUCCESS | All five stages; app version 1 |
| #2 | FAILURE (intentional) | Test stage failed; Deploy and Verify skipped |
| #3, #4 | FAILURE (infrastructure) | Transient GitHub connection error during SCM checkout |
| #5 | SUCCESS | After restarting Jenkins with a short DNS cache TTL; app version 5 |

The application is reachable at `http://localhost:8081` (`/health` returns
`{"status":"ok"}`), and Jenkins at `http://localhost:8080`.

## Report and screenshots (deliverables)

`rapport_TP7_Jenkins.tex` contains the report, the answers to the questions in
Sections 19 and 20, and all screenshot evidence. The repository URL is already set to
`https://github.com/RebahiWassim/jenkins-flask-lab.git`.

Screenshots saved in this folder:

- `jenkins-success.png` — Jenkins Pipeline Steps view of the successful build #5
- `app-running.png` — application in the browser at `http://localhost:8081` (Version: 5)
- `test-failed.png` — failed Test stage console output of build #2
- `jenkins-failed-stages.png` — Pipeline Steps view of build #2 (Test failed, Deploy/Verify skipped)

The `.tex` file automatically inserts an image when the file exists and shows a
labelled placeholder box otherwise.

Compile with: `pdflatex rapport_TP7_Jenkins.tex` (run twice for the table of contents).
