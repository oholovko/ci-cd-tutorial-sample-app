![CI/CD Pipeline](https://github.com/oholovko/ci-cd-tutorial-sample-app/actions/workflows/ci.yml/badge.svg?branch=master)

# CD/CI Tutorial Sample Application ⚙

**NOTE:** This code was written for an
[article](https://medium.com/rockedscience/docker-ci-cd-pipeline-with-github-actions-6d4cd1731030)
in the **RockedScience** publication on Medium.

## Description

This sample Python REST API application was written for a tutorial on implementing Continuous Integration and Delivery pipelines.

It demonstrates how to:

 * Write a basic REST API using the [Flask](http://flask.pocoo.org) microframework
 * Basic database operations and migrations using the Flask wrappers around [Alembic](https://bitbucket.org/zzzeek/alembic) and [SQLAlchemy](https://www.sqlalchemy.org)
 * Write automated unit tests with [unittest](https://docs.python.org/2/library/unittest.html)

Also:

 * How to use [GitHub Actions](https://github.com/features/actions)

## Requirements

 * `Python 3.8`
 * `Pip`
 * `virtualenv`, or `conda`, or `miniconda`

The `psycopg2` package does require `libpq-dev` and `gcc`.
To install them (with `apt`), run:

```sh
$ sudo apt-get install libpq-dev gcc
```

## Installation

With `virtualenv`:

```sh
$ python -m venv venv
$ source venv/bin/activate
$ pip install -r requirements.txt
```

With `conda` or `miniconda`:

```sh
$ conda env create -n ci-cd-tutorial-sample-app python=3.8
$ source activate ci-cd-tutorial-sample-app
$ pip install -r requirements.txt
```

Optional: set the `DATABASE_URL` environment variable to a valid SQLAlchemy connection string. Otherwise, a local SQLite database will be created.

Initalize and seed the database:

```sh
$ flask db upgrade
$ python seed.py
```

## Running tests

Run:

```sh
$ python -m unittest discover
```

## Running the application

### Running locally

Run the application using the built-in Flask server:

```sh
$ flask run
```

### Running on a production server

Run the application using `gunicorn`:

```sh
$ pip install -r requirements-server.txt
$ gunicorn app:app
```

To set the listening address and port, run:

```
$ gunicorn app:app -b 0.0.0.0:8000
```

## Running on Docker

Run:

```
$ docker build -t ci-cd-tutorial-sample-app:latest .
$ docker run -d -p 8000:8000 ci-cd-tutorial-sample-app:latest
```

## Deploying to Heroku

Run:

```sh
$ heroku create
$ git push heroku master
$ heroku run flask db upgrade
$ heroku run python seed.py
$ heroku open
```

or use the automated deploy feature:

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)

## CI/CD Pipeline

The project uses GitHub Actions with the following automated workflows:

| Workflow | Trigger | Purpose |
|---|---|---|
| **CI/CD Pipeline** (`ci.yml`) | Push/PR to `master` | Test → Version → Build → Deploy → Notify |
| **Deploy / Rollback** (`deploy.yml`) | Manual (`workflow_dispatch`) | Deploy or rollback a specific version |
| **Cleanup Old Images** (`cleanup.yml`) | Weekly schedule / Manual | Remove old untagged container images |

### Pipeline Stages

1. **Test** — Lint (flake8) and unit tests (unittest)
2. **Version** — `semantic-release` analyzes commits and creates a new semver tag + GitHub Release
3. **Build** — Docker image built and pushed to GHCR with multiple tags
4. **Deploy** — Deploys the versioned image to production (on `master` push)
5. **Notify** — Sends Slack notification with pipeline status and version info

## Versioning Strategy

This project uses **Semantic Versioning** ([semver.org](https://semver.org)) with the format `MAJOR.MINOR.PATCH`:

- **MAJOR** — breaking changes
- **MINOR** — new features, backwards compatible
- **PATCH** — bug fixes

### How Versions Are Generated

Versions are created automatically by `semantic-release` based on conventional commit messages:

| Commit prefix | Version bump | Example |
|---|---|---|
| `fix:` | Patch (1.0.0 → 1.0.1) | `fix: resolve login error` |
| `feat:` | Minor (1.0.0 → 1.1.0) | `feat: add user profile` |
| `feat!:` or `BREAKING CHANGE:` | Major (1.0.0 → 2.0.0) | `feat!: redesign API` |

When a version is published:
1. A GitHub Release and git tag (`vX.Y.Z`) are created
2. The Docker image is tagged with the version in GHCR

### Docker Image Tags

Each build produces multiple tags in `ghcr.io/oholovko/ci-cd-tutorial-sample-app`:

| Tag | Description |
|---|---|
| `1.2.0` | Semantic version (immutable) |
| `latest` | Most recent build on default branch |
| `master` | Branch name |
| `abc1234` | Short commit SHA |

### Deploy a Specific Version

Go to **Actions** → **Deploy / Rollback** → **Run workflow**, then enter:
- **Version**: the tag to deploy (e.g., `1.0.0` or `latest`)
- **Environment**: `production`, `staging`

### Rollback Procedure

To rollback to a previous version:

1. Go to **Actions** → **Deploy / Rollback** → **Run workflow**
2. Enter the previous version number (e.g., `1.0.0`)
3. Select the target environment
4. Click **Run workflow**

The previous image is pulled directly from GHCR — no rebuild required.

### Artifact Retention

- A scheduled cleanup runs weekly (Sunday 3:00 AM UTC)
- Keeps the **10 most recent** image versions
- Only deletes **untagged** images — semver-tagged versions are always preserved
- Can also be triggered manually from **Actions** → **Cleanup Old Images**

For more information about using Python on Heroku, see these Dev Center articles:

 - [Python on Heroku](https://devcenter.heroku.com/categories/python)
