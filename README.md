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

## Report (deliverable)

`rapport_TP7_Jenkins.tex` contains the report, the answers to the questions in
Sections 19 and 20, and the screenshot evidence slots. Before compiling:

- Replace `USERNAME` in `\repoURL` (top of the `.tex` file) with your GitHub username.
- Save the three screenshots in this folder with these exact names:
  - `jenkins-success.png` — successful Jenkins pipeline (five green stages)
  - `app-running.png` — application in the browser at `http://localhost:8081`
  - `test-failed.png` — failed Test stage in the Jenkins console

The `.tex` file automatically inserts an image when the file exists and shows a
labelled placeholder box otherwise, so it can be compiled at any time.

Compile with: `pdflatex rapport_TP7_Jenkins.tex` (run twice for the table of contents).
