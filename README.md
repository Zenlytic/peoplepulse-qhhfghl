# Zenlytic

[![CI/CD Pipeline (Test on PR, Test and Deploy on Push)](https://github.com/Zenlytic/zenlytic/actions/workflows/test_and_deploy.yml/badge.svg)](https://github.com/Zenlytic/zenlytic/actions/workflows/test_and_deploy.yml)

[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

## Set Up

### Software you'll need:

1. Python 3.11.6
2. [Poetry](https://python-poetry.org/) (Python dependency management) 1.6.1
3. (Optional) [uv](https://docs.astral.sh/uv/) (Faster Python dependency management) 0.3.0
4. Docker (24.0.6 or newer)
5. Docker Compose (2.21.0 or newer)
6. Node v20.16.0 (install node using [nvm](https://github.com/nvm-sh/nvm))
7. Yarn (JavaScript dependency management) (1.22.18 or newer)
8. AWS CLI v2 (see below for instructions)

### Steps to get going:

#### Prerequisites

1. If you're on an M-series Mac, set the environment variables listed [here](#environment-variables-for-m-series-mac-computers) before proceeding.
2. Install [brew](https://brew.sh/). (If you're on an M-series and encounter errors, [follow these instructions](https://stackoverflow.com/questions/66666134/how-to-install-homebrew-on-m1-mac))
3. Download Docker Desktop for Mac ([link here](https://docs.docker.com/get-docker/)). Open Docker Desktop to complete installation.
4. [Install and configure AWS CLI v2.](https://www.notion.so/zenlytics/AWS-CLI-Install-Configure-de398f84aeba44a09756ba0e3de3fe34)
5. Install NVM ([follow these instructions](https://github.com/nvm-sh/nvm))
   - Use NVM to install the Node version found [above](#set-up)
   - Use NVM to install `npm`
   - Install `yarn` globally
     ```
     % npm install -global yarn
     ```
6. Install `poetry` for python dependency management (version found [above](#set-up), link [here](https://python-poetry.org/docs/#installation))
7. Install `postgres`

   ```
   % brew install postgres
   ```

8. Install `libpq`
   ```
   % brew install libpq
   ```

#### The good stuff

1. Clone and enter this repo.
2. Clone the [`metrics_layer`](https://github.com/Zenlytic/metrics_layer) submodule by running `git submodule update --init`.
3. Install python using miniconda’s bash installer ([link here](https://docs.anaconda.com/miniconda/install/#quick-command-line-install)). (Intel processor instructions are behind a toggle)
   - Create a conda environment using the Python version found [above](#set-up). If the version can't be found, try without the patch value, eg. 3.11 instead of 3.11.6
     ```
     % conda create --name zenlytic_py python=<PYTHON_VERSION>
     ```
   - Add the command `conda activate zenlytic_py` to your `.zshrc` file
4. From the `zenlytic` repo root, navigate to `services/back_end/`
   - Run:
     ```
     back_end % python -m venv venv
     ```
   - Then:
     ```
     back_end % source venv/bin/activate
     ```
   - Then:
     ```
     back_end % poetry install
     ```
5. Return to the `zenlytic` repo root and run the following commands to authenticate yourself with AWS to pull docker containers:
   - Configure AWS CLI with the values given to you by Paul
     ```
     zenlytic % aws configure
     ...
     ## The Access Key ID is the first value used to sign into AWS Console UI
     ## Be sure to enter "us-east-1" for "default region name"
     ```
   - Authenticate Docker with AWS credentials
     ```
     zenlytic % aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
     ```
6. Finally, run the following scripts:
   ```
   zenlytic % source ./run-local.sh
   ```
   Then:
   ```
   zenlytic % source ./seed_db.sh
   ```
7. To set the environment variables needed in the front end run `source env-local.sh`, in the terminal window you start the front end server. You should put this in your .bash_profile or .zshrc to reduce manual work.
8. Go to `services/front_end` and run `yarn install` then `yarn start`.
9. Reap the benefits of your labor and go to `http://localhost:3000`. 🎉
10. Ask in slack for the local dev login credentials and try running through the [QA instructions here](https://www.notion.so/zenlytics/How-to-QA-560d0b16f5334487adf316b89df4509a?pvs=4#4c0f369c365646f7be4f397c139eeff5) to test drive your setup.
11. For local development, categorical fields (fields with property `searchable: True` in the data model) are deleted everytime `./seed_db.sh` is run. To test categories for searchable fields, you'll have to refresh Zoe's search index by clicking on your profile icon (top right of the GUI), entering `Workspace Settings > Zoë > Refresh Zoë's search index`.

## Style

Run `pre-commit install` at the root of the repository with the venv active to install the pre-commit hooks. This is a one-time task that will enable auto-validation of every commit locally.

Configure your IDE to lint with `flake8` and to automatically run formatting with `black`. We recommend using [Cursor IDE](https://www.cursor.com/), and the `/.vscode/settings.json` file automatically configures `flake8` and `black` for you. Otherwise, you will need to manually configure the tools. Install any missing packages you need using `pip install <package_name>`. Or, replace `pip` with `uv pip` if `uv` is installed.

To run the linting we run in CI/CD go to the repo root and run `./lint-local.sh`

Any issues will go to `stdout` and you can correct them in your text editor (but there shouldn't be any because you've already set up `black` as your code formatter)

## Testing

To run the tests go to the repo root and run `./test-local.sh`

To get more specific, and only run tests for one file you can run `./test-local.sh functional/test_users.py`

## CI/CD

Our CI/CD runs on GitHub Actions (see `.github/workflows/test_and_deploy.yml`). There are a few things to note:

1. git master branch == the code on [production](https://app.zenlytic.com/)
2. git stage branch == the code on [stage](https://stage.zenlytic.com)
3. every other branch == the code on your [local](http://localhost:3000)

The first step in our CI/CD pipeline after installs is to bring all containers up with `docker compose` and run the tests. If all tests pass, we run the `flake8` linter. If linting passes, we say the commit is good and proceed to build the individual containers. The individual containers (back_end and worker) use two part builds and will cache the first part of the build on AWS ECR to speed up the builds (e.g. the long, boring installs of packages).

We then push the containers, update the definitions, push the new frontend build to S3, and invalidate the Cloudfront cache (only for production deployments). Once that build finishes we tell the service (defined in Terraform) to update the containers running in ECS Fargate with the most recent images.

## Adding dependencies

1. Front end (react): use `yarn add` and `yarn install`
2. Back end (python): add dependency using `poetry add` in the `services/back_end` folder. Then run `./run-local.sh` to rebuild the docker containers with the new dependency.

## Helpful commands

1. To check out the logs for the back end run `docker compose -f docker-compose-local.yml logs | grep back_end`
2. To check out the logs for the worker run `docker compose -f docker-compose-local.yml logs | grep worker_1`
3. To run the same tests we run in CI/CD, run `./test-local.sh`
4. To run the same linting check we run in CI/CD, run `./lint-local.sh`

## Troubleshooting

1. If you get a blank screen when starting the frontend app on the local dev server, with a `204 No Content` response, clear the browser's cache and delete cookies for `http://127.0.0.1:3000/` to ensure the most current version of the app is loaded.

Reach out to Paul to get set up with an AWS account, and get credentials for your development environment

## Environment variables for M-series Mac computers

Add the following entries to your `.zshrc`:

- AWS environment variables (get these values from Paul)
  ```
  # AWS
  export AWS_DEFAULT_REGION=us-east-1
  export AWS_ACCOUNT_ID=734*********
  export AWS_ACCESS_KEY_ID=AKIA*****************
  export AWS_SECRET_ACCESS_KEY=QWDS**************g12e23rf*************************
  ```
- Local development
  ```
  # Zenlytic dev
  export REACT_APP_BACK_END_URL=http://localhost/
  export REACT_APP_FRONT_END_URL=http://localhost:3000
  export REACT_APP_USER_POOL_ID=us-east-1_jsT8Hi9y9
  export BUILD_OS_TYPE="-arm"
  ```
- For `gcc` to properly build packages.

  ```
  # Environment variables - M1 build config
  export LDFLAGS="-L/opt/homebrew/opt/openssl@3/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/openssl@3/include"
  export LDFLAGS="-L/opt/homebrew/opt/libpq/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/libpq/include"
  export PKG_CONFIG_PATH="/opt/homebrew/opt/libpq/lib/pkgconfig"
  export PKG_CONFIG_PATH="/opt/homebrew/opt/openssl@1.1/lib/pkgconfig"
  export PATH="/opt/homebrew/opt/libpq/bin:$PATH"

  export GRPC_PYTHON_BUILD_SYSTEM_OPENSSL=1
  export GRPC_PYTHON_BUILD_SYSTEM_ZLIB=1
  ```

  .
