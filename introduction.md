# Introduction to Continuous Integration/Deployment/Delivery

## Description

## CircleCi - Analogues

|                              | [Circle CI](https://circleci.com/)   | [Travis CI](https://travis-ci.org/) | [Bamboo](https://www.atlassian.com/software/bamboo) | [GitLab CI](https://docs.gitlab.com/ee/ci/)       | [Jenkins](https://jenkins.io/) | [TeamCity](https://www.jetbrains.com/teamcity/) | [Codeship](https://codeship.com/) |
|------------------------------|--------------------------------------|-------------------------------------|-----------------------------------------------------|---------------------------------------------------|--------------------------------|-------------------------------------------------|-----------------------------------|
| Supports CD                  | yes                                  | no                                  | yes                                                 | yes                                               | yes                            | yes                                             | yes                               |
| Cloud hosting                | yes                                  | yes                                 | yes                                                 | yes                                               | yes                            | no                                              | yes                               |
| License                      | Proprietary                          | MIT                                 | Proprietary                                         | MIT/EE                                            | MIT                            | Proprietary                                     | Proprietary                       |
| Paid version price           | $50-3,150 per month                  | $69-489 per month                   | $10-110,000 one-off payment                         | $4-99 per month                                   | -                              | $299-21,999 one-off payment                     | $75-1,500 per month               |
| Free version                 | yes                                  | yes                                 | yes                                                 | yes                                               | yes                            | yes                                             | yes                               |
| Special feature              | Easy to use                          | Direct connectivity with GitHub     |                                                     | Awesome Docker support | A lot of plugins               | Gated commits                                   | Pro & Basic versions              |
| Docs and support             | Good                                 | Poor                                | Good                                                | Good                                              | Adequate                       | Good                                            | Poor                              |
| Learning curve and usability | Easy                                 | Easy                                | Medium                                              | Easy                                              | Easy                           | Medium                                          | Easy                              |
| Use case                     | For fast development and high budget | For small projects and startups     | For Atlassian integrations                          | For any project                                   | For big projects               | For enterprise needs                            | For any project                   |

## Pros and Cons

#### Main CircleCI strength

**Simple UI.** CircleCI is recognized for its user-friendly interface for managing builds/jobs. Its single-page web app is clean and easy to understand
**High-quality customer support.** StackShare’s community members highlight CircleCI’s speedy support: They respond to requests within 12 hours.
CircleCI runs **all types of software tests** including web, mobile, and container environments.
**Caching of requirements installation and third-party dependencies.** Instead of installing the environments, CircleCI can take data from multiple projects using the granular check-out-key options.
**No need for manual debugging.** CircleCI has debugging feature Debug via SSH, but Jenkins users have to debug manually by clicking on jobs.

## Executors and Images

#### Overview

CircleCI enables you to run jobs in one of three environments:

* Within Docker images (docker)
* Within a Linux virtual machine (VM) image (machine)
* Within a macOS VM image (macos)

It is possible to specify a different executor type for every job in your `.circleci/config.yml` by specifying the 
executor type and an appropriate image.
Main differences between using Docker and VM:

| Virtual Environment                        | Docker  | Machine   |
|--------------------------------------------|---------|-----------|
| Start time                                 | Instant | 30-60 sec |
| Clean environment                          | Yes     | Yes       |
| Custom images                              | Yes     | No        |
| Build Docker images                        | Yes     | Yes       |
| Full control over job environment          | No      | Yes       |
| Full root access                           | No      | Yes       |
| Run multiple databases                     | Yes     | Yes       |
| Run multiple versions of the same software | No      | Yes       |
| Layer caching                              | Yes     | Yes       |
| Run privileged containers                  | No      | Yes       |
| Use docker compose with volumes            | No      | Yes       |
| Configurable resources (CPU/RAM)           | Yes     | No        |

The docker key defines Docker as the underlying technology to run your jobs using Docker Containers. 
Containers are an instance of the Docker Image you specify. The first image listed in your configuration is 
the primary container image in which all steps run. All other containers run in a common network and every exposed 
port will be available on localhost from a primary container.

#### Pre-Built CircleCI Docker Images

For convenience, CircleCI maintains several Docker images. These images are typically extensions of official Docker 
images and include tools especially useful for CI/CD. All of these pre-built images are available in the [CircleCI org
on Docker Hub](https://hub.docker.com/search?q=circleci&type=image). 

CircleCI’s convenience images fall into two categories: **language images** and **service images**. 
All images add a circleci user as a system user.

#### Language Images

Language images are convenience images for common programming languages. These images include both the relevant 
language and commonly-used tools. A language image should be listed first under the docker key in your configuration, 
making it the primary container during execution.

CircleCI maintains images for the languages below.

* Android
* Clojure
* Elixir
* Go (Golang)
* JRuby
* Node.js
* OpenJDK (Java)
* PHP
* Python
* Ruby
* Rust

#### Language Image Variants

CircleCI maintains several variants for language images. To use these variants, add one of the following suffixes 
to the end of an image tag.

* `-node` includes Node.js for polyglot applications
* `-browsers` includes Chrome, Firefox, Java 8, and Geckodriver
* `-browsers-legacy` includes Chrome, Firefox, Java 8, and PhantomJS
* `-node-browsers` combines the -node and -browsers variants
* `-node-browsers-legacy` combines the -node and -browsers-legacy variants

#### Service Images

Service images are convenience images for services like databases. These images should be listed after language images 
so they become secondary service containers.

CircleCI maintains images for the services below.

* buildpack-deps
* DynamoDB
* MariaDB
* MongoDB
* MySQL
* PostgreSQL
* Redis

#### Sample Configuration

```yml
version: 2
jobs:
  build:
    docker:
      - image: circleci/ruby:2.5.1-node
      - image: circleci/postgres:9.6
      - image: redis
```


## Environment variables

Environment variables are used according to a specific precedence order, as follows:

1. Environment variables declared inside a shell command in a run step, for example FOO=bar make install.
2. Environment variables declared with the environment key for a run step.
3. Environment variables set with the environment key for a job.
4. Environment variables set with the environment key for a container.
5. Context environment variables (assuming the user has access to the Context).
6. Project-level environment variables set on the Project Settings page.
7. Special CircleCI environment variables.

**Note:** Do not add secrets or keys inside the `.circleci/config.yml` file. The full text of `config.yml` is visible to 
developers with access to your project on CircleCI. Store secrets or keys in project or context settings 
in the CircleCI app.

#### Setting an Environment Variable in a Shell Command

While CircleCI does not support interpolation when setting environment variables, it is possible to set variables for 
the current shell by using BASH_ENV. This is useful for both modifying your PATH and setting environment variables 
that reference other variables.

In every step, CircleCI uses bash to source BASH_ENV. This means that BASH_ENV is automatically loaded and run, 
allowing you to use interpolation and share environment variables across run steps.

```yml
version: 2
jobs:
  build:
    docker:
      - image: smaant/lein-flyway:2.7.1-4.0.3
    steps:
      - run:
          name: Update PATH and Define Environment Variable at Runtime
          command: |
            echo 'export PATH=/path/to/foo/bin:$PATH' >> $BASH_ENV
            echo 'export VERY_IMPORTANT=$(cat important_value)' >> $BASH_ENV
            source $BASH_ENV
```

#### Setting an Environment Variable in a Step, Job, Container
To set an environment variable use the environment key.

```yml
version: 2
jobs:
  build:
    docker:
      - image: smaant/lein-flyway:2.7.1-4.0.3
      - image: circleci/postgres:9.6-jessie
      # environment variables for all commands executed in the primary container
        environment:
          POSTGRES_USER: conductor
          POSTGRES_DB: conductor_test
  # environment variables for all commands executed in the job
    environment:
      JOB_VARIABLE: 'foo'
    steps:
      - checkout
      - run:
          name: Run migrations
          command: sql/docker-entrypoint.sh sql
          # Environment variable for a single command shell
          environment:
            DATABASE_URL: postgres://conductor:@localhost:5432/conductor_test
```

#### Setting an Environment Variable in a Project

1. In the CircleCI application, go to your project’s settings by clicking the gear icon next to your project.
2. In the **Build Settings** section, click on **Environment Variables**.
3. Import variables from another project by clicking the **Import Variable(s)** button. Add new variables by clicking 
the **Add Variable** button.
4. Use your new environment variables in your `.circleci/config.yml` file.

Once created, environment variables are hidden and uneditable in the application. Changing an environment variable 
is only possible by deleting and recreating it.

#### Injecting Environment Variables with the API

Build parameters are environment variables, therefore their names have to meet the following restrictions:

* They must contain only ASCII letters, digits and the underscore character.
* They must not begin with a number.
* They must contain at least one character.

Aside from the usual constraints for environment variables there are no restrictions on the values themselves and 
are treated as simple strings. The order that build parameters are loaded in is not guaranteed so avoid interpolating 
one build parameter into another. It is best practice to set build parameters as an unordered list of independent 
environment variables.

```bash
curl \
  --header "Content-Type: application/json" \
  --data '{"build_parameters": {"param1": "value1", "param2": 500}}' \
  --request POST \
  https://circleci.com/api/v1.1/project/github/circleci/mongofinil/tree/master?circle-token=$CIRCLE_TOKEN
```

In the above example, `$CIRCLE_TOKEN` is a personal API token.

#### Built-in Environment Variables

| Variable                  | Type    | Value                                                                                                                                     |
|---------------------------|---------|-------------------------------------------------------------------------------------------------------------------------------------------|
| CI                        | Boolean | true (represents whether the current environment is a CI environment)                                                                     |
| CI_PULL_REQUEST           | String  | Deprecated version of CIRCLE_PULL_REQUEST. Kept for backward compatibility with CircleCI 1.0.                                             |
| CI_PULL_REQUESTS          | List    | Deprecated version of CIRCLE_PULL_REQUESTS. Kept for backward compatibility with CircleCI 1.0.                                            |
| CIRCLE_BRANCH             | String  | The name of the Git branch currently being built.                                                                                         |
| CIRCLE_BUILD_NUM          | Integer | The number of the CircleCI build.                                                                                                         |
| CIRCLE_BUILD_URL          | String  | The URL for the current build.                                                                                                            |
| CIRCLE_COMPARE_URL        | String  | The GitHub or Bitbucket URL to compare commits of a build.                                                                                |
| CIRCLE_INTERNAL_TASK_DATA | String  | The directory where test timing data is saved.                                                                                            |
| CIRCLE_JOB                | String  | The name of the current job.                                                                                                              |
| CIRCLE_NODE_INDEX         | Integer | The index of the specific build instance. A value between 0 and (CIRCLECI_NODE_TOTAL - 1)                                                 |
| CIRCLE_NODE_TOTAL         | Integer | The number of total build instances.                                                                                                      |
| CIRCLE_PR_NUMBER          | Integer | The number of the associated GitHub or Bitbucket pull request. Only available on forked PRs.                                              |
| CIRCLE_PR_REPONAME        | String  | The name of the GitHub or Bitbucket repository where the pull request was created. Only available on forked PRs.                          |
| CIRCLE_PR_USERNAME        | String  | The GitHub or Bitbucket username of the user who created the pull request. Only available on forked PRs.                                  |
| CIRCLE_PREVIOUS_BUILD_NUM | Integer | The number of previous builds on the current branch.                                                                                      |
| CIRCLE_PROJECT_REPONAME   | String  | The name of the repository of the current project.                                                                                        |
| CIRCLE_PROJECT_USERNAME   | String  | The GitHub or Bitbucket username of the current project.                                                                                  |
| CIRCLE_PULL_REQUEST       | String  | The URL of the associated pull request. If there are multiple associated pull requests, one URL is randomly chosen.                       |
| CIRCLE_PULL_REQUESTS      | List    | Comma-separated list of URLs of the current build’s associated pull requests.                                                             |
| CIRCLE_REPOSITORY_URL     | String  | The URL of your GitHub or Bitbucket repository.                                                                                           |
| CIRCLE_SHA1               | String  | The SHA1 hash of the last commit of the current build.                                                                                    |
| CIRCLE_TAG                | String  | The name of the git tag, if the current build is tagged. For more information, see the Git Tag Job Execution.                             |
| CIRCLE_USERNAME           | String  | The GitHub or Bitbucket username of the user who triggered the build.                                                                     |
| CIRCLE_WORKFLOW_ID        | String  | A unique identifier for the workflow instance of the current job. This identifier is the same for every job in a given workflow instance. |
| CIRCLE_WORKING_DIRECTORY  | String  | The value of the working_directory key of the current job.                                                                                |
| CIRCLECI                  | Boolean | true (represents whether the current environment is a CircleCI environment)                                                               |
| HOME                      | String  | Your home directory.                                                                                                                      |

#### Best Practices
The best way to store environment variables is keeping them in the repository. In this way you avoid problems with 
absent or wrong variables. 

Variables, that needed only on CircleCI or secret variables, should be stored in CircleCi 
Project Setting page.

## Caching
## Containers/Parallelism/Jobs
## Project configurations
## Steps
## Jobs
## Workflows
