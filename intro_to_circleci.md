# Introduction to CircleCi

**CircleCI** is a continuous integration service for web and mobile applications.
One of the advantages of this service is the build in the cloud, that is, you do not need a local build machine. The service allows you to flexibly configure assembly testing, as the user has access to ```sudo```.

CircleCI integrates with a VCS and automatically runs a series of steps every time that it detects a change to your repository.

A CircleCI build consists of a series of steps. Generally, they’re:

- Dependencies
- Testing
- Deployment

## CircleCi - Analogues

|                              | [Circle CI](https://circleci.com/)   | [Travis CI](https://travis-ci.org/) | [Bamboo](https://www.atlassian.com/software/bamboo) | [GitLab CI](https://docs.gitlab.com/ee/ci/)       | [Jenkins](https://jenkins.io/) | [TeamCity](https://www.jetbrains.com/teamcity/) | [Codeship](https://codeship.com/) | [GitHub Actions](https://github.com/features/actions) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Supports CD | yes | no | yes | yes | yes | yes | yes | yes |
| Cloud hosting | yes | yes | yes | yes | yes | no | yes | yes |
| Paid version price | $50-3,150 per month | $69-489 per month | $10-110,000 one-off payment | $4-99 per month | - | $299-21,999 one-off payment | $75-1,500 per month |  $0-4,380 per month |
| Docs and support | Good | Poor | Good | Good | Adequate | Good | Poor | Adequate |
| Learning curve and usability | Easy | Easy | Medium | Easy | Easy | Medium | Easy | Easy |
| Use case | For fast development and high budget | For small projects and startups | For Atlassian integrations |For any project | For big projects | For enterprise needs | For any project | For any project |

## Pros and Cons

### Pros:

- **Generous free plans** for open source projects easy to set up plenty of features plenty of customization runs quick

- **High-quality customer support.** StackShare’s community members highlight CircleCI’s speedy support: They respond to requests within 12 hours.

- CircleCI runs **all types of software tests** including web, mobile, and container environments.

- **Caching of requirements installation and third-party dependencies.** Instead of installing the environments, CircleCI can take data from multiple projects using the granular check-out-key options.

- **No need for manual debugging.** CircleCI has debugging feature Debug via SSH, but Jenkins users have to debug manually by clicking on jobs.

- ability to **local** setup CircleCi

### Cons:

- Config file ```.circleci/config.yml``` tends to get **too long** 

- **High price** for private repositories and large projects

- You can put **only one** language image in a container

- CircleCI in the free version **only** supports Ubuntu. To use MacOS or Windows you will have to pay

- The local version has a **limitation** compared to the cloud CircleCi

- **Shared containers** between all projects

- **Not understandable** UI/UX interface in most cases

- **Strange** system of approvable deploys

## Executors and Images

### Overview

CircleCI enables you to run jobs in one of four environments:

* Within Docker images (docker)
* Within a Linux virtual machine (VM) image (machine)
* Within a macOS VM image (macos)
* Within a Windows VM image

It is possible to specify a different executor type for every job in your `.circleci/config.yml` by specifying the 
executor type and an appropriate image.
Main differences between using Docker and VM:

| Virtual Environment                        | Docker  | Machine   |
|--------------------------------------------|---------|-----------|
| Start time                                 | Instant | 30-60 sec |
| Clean environment                          | Yes     | Yes       |
| Custom images                              | Yes     | No        |
| Full control over job environment          | No      | Yes       |
| Full root access                           | No      | Yes       |
| Run multiple databases                     | Yes     | Yes       |
| Run multiple versions of the same software | No      | Yes       |
| Layer caching                              | Yes     | Yes       |
| Run privileged containers                  | No      | Yes       |
| Use docker compose with volumes            | No      | Yes       |
| Configurable resources (CPU/RAM)           | Yes     | No        |

The docker key defines Docker as the underlying technology to run your jobs using Docker Containers.

Containers are an instance of the Docker Image you specify. The first image listed in your configuration is the primary container image in which all steps run. All other containers run in a common network and every exposed port will be available on localhost from a primary container.

### Pre-Built CircleCI Docker Images

For convenience, CircleCI maintains several Docker images. These images are typically extensions of official Docker 
images and include tools especially useful for CI/CD. All of these pre-built images are available in the [CircleCI org
on Docker Hub](https://hub.docker.com/search?q=circleci&type=image). 

CircleCI’s convenience images fall into two categories: **language images** and **service images**. 
All images add a circleci user as a system user.

### Language Images

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

### Language Image Variants

CircleCI maintains several variants for language images. To use these variants, add one of the following suffixes 
to the end of an image tag.

* `-node` includes Node.js for polyglot applications
* `-browsers` includes Chrome, Firefox, Java 8, and Geckodriver
* `-browsers-legacy` includes Chrome, Firefox, Java 8, and PhantomJS
* `-node-browsers` combines the -node and -browsers variants
* `-node-browsers-legacy` combines the -node and -browsers-legacy variants

### Service Images

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

### Sample Configuration

```yml
version: 2
jobs:
  build:
    docker:
      - image: circleci/ruby:2.6.1-node
      - image: circleci/postgres:11.5
      - image: redis
```

### Best Practices

* Do **not** use `latest`. Use the most specific image possible. This makes your builds more deterministic by preventing an upstream image from introducing unintended changes to your image. 
There are two ways to make an image more specific:

  * Use a tag to pin an image to a version or operating system (OS).
  for example:
    ```yml
    #instead of 
    - image: circleci/ruby

    #use
    - image: circleci/ruby:2.6.1-node
    ```
  * Use a Docker image ID to pin an image to a fixed version. for example:
    ```yml
    - image: circleci/ruby@sha256:df1808e61a9c32d0ec110960fed213ab2339451ca88941e9be01a03adc98396e
    ```

  ####  Finding an Image ID
  1. In the CircleCI application, go to a past build that used the image.
  2. On the `Test Summary` tab, click the `Spin up environment` step.
  3. In the log output, locate the digest for the image.
  4. Add the image ID to the image name as shown below.




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

### Setting an Environment Variable in a Shell Command

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
**Note:** Depending on your shell, you may have to append the new variable to a shell startup file like `~/.tcshrc` or `~/.zshrc`

### Setting an Environment Variable in a Step, Job, Container
To set an environment variable use the environment key.

```yml
version: 2
jobs:
  build:
    docker:
      - image: smaant/lein-flyway:2.7.1-4.0.3
      - image: circleci/postgres:9.6-jessie
      # environment variables for the container in which were defined
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

### Setting an Environment Variable in a Project

1. In the CircleCI application, go to your project’s settings by clicking the gear icon next to your project.
2. In the **Build Settings** section, click on **Environment Variables**.
3. Import variables from another project by clicking the **Import Variable(s)** button. Add new variables by clicking 
the **Add Variable** button.
4. Use your new environment variables in your `.circleci/config.yml` file.

Once created, environment variables are hidden and uneditable in the application. Changing an environment variable 
is only possible by deleting and recreating it.

### Setting an Environment Variable in a Context
Creating a context allows you to share environment variables across multiple projects. To set an environment variables in a context, see the [Contexts documentation](https://circleci.com/docs/2.0/contexts/).

### Built-in Environment Variables
  The **built-in** environment variables are exported in each build and can be used for more complex testing or deployment.  
  
  **Note:** You cannot use a built-in environment variable to define another environment variable. Instead, you must use a run step to export the new environment variables using BASH_ENV

  The list of buil-in variables you can see [here](https://circleci.com/docs/2.0/env-vars/#built-in-environment-variables)
### Best Practices
* The best way to store environment variables is keeping them in the repository. In this way you avoid problems with 
absent or wrong variables. 

* Variables, that needed only on CircleCI or secret variables, should be stored in CircleCi 
Project Setting page.

* We recommend keeping cache sizes **under 500MB**. This is our upper limit for corruption checks because above this limit check times would be excessively long. You can view the cache size from the CircleCI Jobs page within the `restore_cache` step. 

## Caching

Caching is one of the most effective ways to make jobs faster on CircleCI by reusing the data from expensive fetch operations from previous jobs.

A good example is package dependency managers such as Yarn, Bundler, or Pip. With dependencies restored from a cache, commands like yarn install will only need to download new dependencies, if any, and not redownload everything on every build.

Automatic dependency caching is not available in CircleCI 2.0, so it is important to plan and implement your caching strategy to get the best performance. Manual configuration in 2.0 enables more advanced strategies and finer control.

**Note:** The Docker images used for CircleCI 2.0 job runs are automatically cached on the server infrastructure where possible.

Sometimes for large projects it’s beneficial to cache the git repository. Here is an example of source caching:
```
    steps:
      - restore_cache:
          keys:
            - source-v1-{{ .Branch }}-{{ .Revision }}
            - source-v1-{{ .Branch }}-
            - source-v1-

      - checkout

      - save_cache:
          key: source-v1-{{ .Branch }}-{{ .Revision }}
          paths:
            - ".git"
```
### Cache Expiration
 The caches created via the save_cache step are stored for up to **30 days**.

### Clearing Cache
For example, you may want to clear the cache in the following scenarios by incrementing the cache key name:

- Dependency manager version change, for example, you change npm from 4 to 5
- Language version change, for example, you change ruby 2.3 to 2.4
- Dependencies are removed from your project

In order to clear the cache, you need to change the name of the key under which it is stored for example:
```
  - v1-npm-deps-{{ checksum "package-lock.json" }} 
```
  change to
```
  - v2-npm-deps-{{ checksum "package-lock.json" }}
```

### Keys and Templates

A cache stores a hierarchy of files under a key.
Following are keys examples for different goals:
`myapp-{{ checksum "package-lock.json" }}` - Cache will be regenerated every time something is changed in package-lock.json file, different branches of this project will generate the same cache key.
`myapp-{{ .Branch }}-{{ checksum "package-lock.json" }}` - Cache will be regenerated every time something is changed in package-lock.json file, different branches of this project will generate separate cache keys.
`myapp-{{ epoch }}` - Every build will generate separate cache keys.
During step execution, the templates above will be replaced by runtime values and use the resultant string as the key.

The following table describes the available cache key templates:

| **Template**                        | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|---------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| {{ .Branch }}                   | The VCS branch currently being built.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| {{ .BuildNum }}                 | The CircleCI job number for this build.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| {{ .Revision }}                 | The VCS revision currently being built.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| {{ .Environment.variableName }} | The environment variable variableName (supports any environment variable exported by CircleCI or added to a specific Context—not any arbitrary environment variable).                                                                                                                                                                                                                                                                                                      |
| {{ checksum "filename" }}       | A base64 encoded SHA256 hash of the given filename’s contents, so that a new cache key is generated if the file changes. This should be a file committed in your repo. Consider using dependency manifests, such as package-lock.json, pom.xml or project.clj. The important factor is that the file does not change between restore_cache and save_cache, otherwise the cache will be saved under a cache key that is different from the file used at restore_cache time. |
| {{ epoch }}                     | The number of seconds that have elapsed since 00:00:00 Coordinated Universal Time (UTC), also known as POSIX or Unix epoch.                                                                                                                                                                                                                                                                                                                                                |
| {{ arch }}                      | Captures OS and CPU (architecture, family, model) information. Useful when caching compiled binaries that depend on OS and CPU architecture, for example, darwin-amd64-6_58 versus linux-amd64-6_62.                                                                                                                                                                                                                                      |

### Full Example of Saving and Restoring Cache

```
steps:
  - checkout

  # Run bundler
  # Load installed gems from cache if possible, bundle install then save cache
  # Multiple caches are used to increase the chance of a cache hit

  - restore_cache:
      keys:
        # CircleCI restores caches in the order of keys listed in the restore_cache step.
        # Each cache key is namespaced to the project, and retrieval is prefix-matched.
        # The cache will be restored from the first matching key.
        - gem-cache-v1-{{ arch }}-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
        - gem-cache-v1-{{ arch }}-{{ .Branch }}
        - gem-cache-v1

  - run: bundle install --path vendor/bundle

  - save_cache:
      key: gem-cache-v1-{{ arch }}-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
      paths:
        - vendor/bundle

  # Precompile assets
  # Load assets from cache if possible, precompile assets then save cache
  # Multiple caches are used to increase the chance of a cache hit

  - restore_cache:
      keys:
        - asset-cache-v1-{{ arch }}-{{ .Branch }}-{{ .Environment.CIRCLE_SHA1 }}
        - asset-cache-v1-{{ arch }}-{{ .Branch }}
        - asset-cache-v1

  - run: bundle exec rake assets:precompile

  - save_cache:
      key: asset-cache-v1-{{ arch }}-{{ .Branch }}-{{ .Environment.CIRCLE_SHA1 }}
      paths:
        - public/assets
        - tmp/cache/assets/sprockets

  - run: bundle exec rspec
```

### Best Practices
- If your source code changes frequently, use fewer, more specific keys. This produces a more granular source cache that will update more often as the current branch and git revision change. E.g. `source-v1-{{ .Branch }}-{{ .Revision }}`
- Use checksum for lockfiles (for example, `Gemfile.lock` or `yarn.lock`)


## Containers

A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings

Container images become containers at runtime and in the case of Docker containers - images become containers when they run on [Docker Engine](https://www.docker.com/products/container-runtime) . Available for both Linux and Windows-based applications, containerized software will always run the same, regardless of the infrastructure. Containers isolate software from its environment and ensure that it works uniformly despite differences for instance between development and staging.

Every change committed to your version control system triggers CircleCI to checkout your code and run your job workflow inside a fresh, on-demand, isolated container with access to the following, depending on your plan:

- **Concurrency** - Utilizing multiple containers to run multiple builds at the same time. To take advantage of concurrency, configure your development workflow using the [Orchestrating Workflows document](https://circleci.com/docs/2.0/workflows/) and run your jobs in parallel as shown in the [Sample 2.0 Config Files document](https://circleci.com/docs/2.0/sample-config/#sample-configuration-with-parallel-jobs).

- **Parallelism** - Splitting tests across multiple containers, allowing you to dramatically speed up your test suite. Update your ```.circleci/config.yml``` file to run your tests in parallel as described in the [Configuring CircleCI](https://circleci.com/docs/2.0/configuration-reference/#parallelism) document. Learn how to update your config file to parallelize and split tests to decrease your build time by reading the [Running Tests in Parallel](https://circleci.com/docs/2.0/parallelism-faster-jobs/) documentation.



## Parallelism
If your project has a large number of tests, it will need more time to run them on one machine. To reduce this time, you can run tests in parallel by spreading them across multiple machines. This requires specifying a parallelism level. You can use either the CircleCI CLI to split test files, or use environment variables to configure each parallel machine individually.

### Specifying a Job’s Parallelism Level
Test suites are conventionally defined at the job level in your ```.circleci/config.yml``` file. The ```parallelism``` key specifies how many independent executors will be set up to run the steps of a job.

To run a job’s steps in parallel, set the ```parallelism``` key to a value greater than 1.

```yml
# ~/.circleci/config.yml
version: 2
jobs:
  test:
    docker:
      - image: circleci/<language>:<version TAG>
    parallelism: 4
```

## Project configurations
CircleCI provides Project and Org settings with encrypted storage in the CircleCI app.

### Overview
To support the open source community, projects that are public on GitHub or Bitbucket receive three free build containers, for a total of four containers. Only **one** container is available for private repositories

### Private Environment Variables
Many projects require API tokens, SSH keys, or passwords. Private environment variables allow you to safely store secrets, even if your project is public. For more information, see the [Environment Variables](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project) document.

### Only Build Pull Requests
By default, CircleCI builds every commit from every branch. This behavior may be too aggressive for open source projects, which often have significantly more commits than private projects. To change this setting, go to the **Advanced Settings** of your project and set the **Only build pull requests** option to On.

**Note:** Even if this option is enabled, CircleCI will still build all commits from your project’s default branch.

### Auto-cancel redundant builds
automatically closes the build if there is a **newer**.

Pipelines must be enabled in order to use this feature.

### Recommended configurations:
- [x] Auto-cancel redundant builds
- [x] Only build pull requests   
- [x] GitHub Status updates
- [x] Enable pipelines
  

## Steps

Steps are a collection of executable commands which are run during a job. The steps setting in a job should be a list of single key/value pairs, the key of which indicates the step type. The value may be either a configuration map or a string (depending on what that type of step requires).

Example, using a map:
``` yml
jobs:
  build:
    steps:
      - run:
          name: Running tests
          command: make test
```

Example, using a string (name here will have the same value as command):

```yml
jobs:
  build:
    steps:
      - run: make test
```

Another shorthand, which is possible for some steps, is to simply use the step name as a string instead of a key/value pair:

```yml
jobs:
  build:
    steps:
      - checkout
```

Each built-in (default) step type is described below.

- **```run```**

  Used for invoking all command-line programs. Each run declaration represents a new shell. It has several attributes:
    - ```command``` - command to execute.
      It’s possible to specify a multi-line ```command```:

        ```yml
        - run:
            command: |
              echo Running test
              mkdir -p /tmp/test-results
              make test
        ```

    - ```name``` - title of the step to be shown in UI
    - ```shell``` - shell to use for execution command
    - ```environment``` - additional environmental variables, locally scoped to command
    - ```background``` - configures commands to run in the background (default: false)

      The following example shows the config for running feature tests. Here frontend/backend servers start and wait in the background till tests will be launched:

      ```yml
      jobs:
        cypress:
          steps:
            - checkout
            - run:
                name: Cloning backend repository
                command: git clone -b develop git@github.com:rubygarage/backend.git
            - run:
                name: Installing gems
                command: bundle install --path vendor/bundle
            - run:
                name: Creating database
                command: bundle exec rails db:create
            - run:
                name: Starting backend server
                command: bundle exec rails s
                background: true
            - run:
                name: Build frontend server
                command: yarn build
            - run:
                name: Starting frontend server
                command: yarn start
                background: true
            - run:
                name: Waiting for frontend server
                command: yarn wait-on http://localhost:4000/login -t 20000
            - run:
                name: Running cypress tests
                command: yarn cypress run
      ```

    - ```when``` - specify when to enable or disable the step. Takes the following values: ```always```, ```on_success```, ```on_fail``` (default: ```on_success```)

    - ```no_output_timeout``` - elapsed time the command can run without output (default: 10 minutes)
    - ```working_directory``` - in which directory to run this step (default: working_directory of the job)

- **```when```** (requires version: 2.1)

  A conditional step consists of a step with the key ```when``` or ```unless```. The purpose of the ```when``` step is customizing commands and job configuration to run on custom conditions (determined at config-compile time) that are checked before a workflow runs. Under the ```when``` key are the subkeys:
    - ```condition``` - a parameter value
    - ```steps``` - a list of steps to execute when the condition is true

    ```yml
    version: 2.1
    jobs: # conditional steps may also be defined in `commands:`
      myjob:
        parameters:
          preinstall-foo:
            type: boolean
            default: false
        machine: true
        steps:
          - run: echo "preinstall is << parameters.preinstall-foo >>"
          - when:
              condition: << parameters.preinstall-foo >>
              steps:
                - run: echo "preinstall"
          - unless:
              condition: << parameters.preinstall-foo >>
              steps:
                - run: echo "don't preinstall"
    workflows:
      workflow:
        jobs:
          - myjob:
              preinstall-foo: false
          - myjob:
              preinstall-foo: true
    ```

- **```checkout```**

  Special step used to check out source code to the configured ```path``` (defaults to the ```working_directory```). The reason this is a special step is because it is more of a helper function designed to make checking out code easy for you. If ```path``` already exists and is
  a git repo - step will not clone whole repo, instead will pull origin, otherwise the step will fail.

- **```setup_remote_docker```**

  Creates a remote Docker environment configured to execute Docker commands.

- **```save_cache```**

- **```restore_cache```**

- **```deploy```**

  Special step for deploying artifacts. ```deploy``` step behaves and uses the same configuration map and semantics as ```run``` step. Jobs may have more than one ```deploy``` step.

- **```store_artifacts```**

  Step to store artifacts (for example logs, binaries, failed feature tests screenshots etc) to be available in the web app or through the API. Subkeys:

    - ```path``` - directory in the primary container to save as job artifacts
    - ```destination``` - prefix added to the artifact paths in the artifacts API (default: the directory of the file specified in ```path```)

- **```store_test_results```**

  Special step used to upload and store test results for a build. Test results are visible on the CircleCI web application, under each build’s “Test Summary” section. Subkey:

  - ```path``` - path (absolute, or relative to your ```working_directory```) to directory containing subdirectories of JUnit XML or Cucumber JSON test metadata files

- **```persist_to_workspace```**

  Special step used to persist a temporary file to be used by another job in the workflow. Workspaces are stored for up to 30 days after being created. Subkeys:

  - ```root``` - either an absolute path or a path relative to ```working_directory```
  - ```paths``` - glob identifying file(s), or a non-glob path to a directory to add to the shared workspace. Interpreted as relative to the workspace root

- **```attach_workspace```**

  Special step used to attach the workflow’s workspace to the current container. The full contents of the workspace are downloaded and copied into the directory the workspace is being attached at. Subkey:

  - ```at``` - directory to attach the workspace to

- **```add_ssh_keys```**

  Special step that adds SSH keys from a project’s settings to a container. Also configures SSH to use these keys. Subkey:

  - ```fingerprints``` - list of fingerprints corresponding to the keys to be added (default: all keys added)

### Best Practices
Often steps may repeating. What could be done to follow DRY principle is using YAML anchors & aliases. For example:

```yml
references:
  restore_bundle_cache: &restore_bundle_cache
    restore_cache:
      keys:
        - repo-bundle-v2-{{ checksum ".ruby-version" }}-{{ checksum "Gemfile.lock" }}

  bundle_install: &bundle_install
    run:
      name: Installing gems
      command: bundle install --path vendor/bundle

  save_bundle_cache: &save_bundle_cache
    save_cache:
      key: repo-bundle-v2-{{ checksum ".ruby-version" }}-{{ checksum "Gemfile.lock" }}
      paths:
        - vendor/bundle
```

```yml
steps:
  - <<: *restore_bundle_cache
  - <<: *bundle_install
  - <<: *save_bundle_cache
```

## Jobs

**Job** is a collection of Steps. All of the steps in the job are executed in a single unit which consumes a CircleCI container from your plan while it’s running.

A run is comprised of one or more named jobs. Jobs are specified in the ```jobs``` map, see [Sample 2.0 config.yml](https://circleci.com/docs/2.0/sample-config/) for two examples of a job map. The name of the job is the key in the map, and the value is a map describing the job.

If you are using [Workflows](https://circleci.com/docs/2.0/workflows/), jobs must have a name that is unique within the ```.circleci/config.yml``` file.

![ci](images/workflow_diagram.png)

If you are **not** using workflows, the jobs map must contain a job named ```build```. This ```build``` job is the default entry-point for a run that is triggered by a push to your VCS provider. It is possible to then specify additional jobs and run them using the CircleCI API.

**Note:** Jobs have a maximum runtime of 5 hours. If your jobs are timing out, consider running some of them in parallel.

Each job consists of the job’s name as a key and a map as a value. A name should be unique within a current jobs list. The value map has the following attributes:

| **key**       | **Required**   | **Type**  | **Description** 
| --- | ---| ---| ---|
| docker | Y | List | Options for [docker executor](https://circleci.com/docs/2.0/configuration-reference/#docker) |
| machine | Y | Map | Options for [machine executor](https://circleci.com/docs/2.0/configuration-reference/#machine)|
| macos | Y | Map | Options for [macOS executor](https://circleci.com/docs/2.0/configuration-reference/#macos) |
| shell | N | String | Shell to use for execution command in all steps. Can be overridden by shell in each step (default: See [Default Shell Options](https://circleci.com/docs/2.0/configuration-reference/#default-shell-options))|
| steps | N | List | A list of steps to be performed |
| working_directory | N | String | In which directory to run the steps. Default: ```~/project``` (where ```project``` is a literal string, not the name of your specific project). Processes run during the job can use the ```$CIRCLE_WORKING_DIRECTORY``` environment variable to refer to this directory. **Note**: Paths written in your YAML configuration file will not be expanded; if your ```store_test_results.path``` is ```$CIRCLE_WORKING_DIRECTORY/tests```, then CircleCI will attempt to store the ```test``` subdirectory of the directory literally named ```$CIRCLE_WORKING_DIRECTORY```, dollar sign ```$``` and all. |
| parallelism | N | Integer | Number of parallel instances of this job to run (default: 1) |
| environment | N | Map | A map of environment variable names and values. |
| branches | N | Map | 	A map defining rules for allow/block execution of specific branches for a single job that is not in a workflow or a 2.1 config (default: all allowed). See [Workflows]() for configuring branch execution for jobs in a workflow or 2.1 config. |

## Workflows
A **workflow** is a set of rules for defining a collection of jobs and their run order. Workflows support complex job orchestration using a simple set of configuration keys to help you resolve failures sooner.

With workflows, you can:

- Run and troubleshoot jobs independently with real-time status feedback.
- Schedule workflows for jobs that should only run periodically.
- Fan-out to run multiple jobs in parallel for efficient version testing.
- Fan-in to quickly deploy to multiple platforms.

For example, if only one job in a workflow fails, you will know it is failing in real-time. Instead of wasting time waiting for the entire build to fail and rerunning the entire job set, you can rerun just the failed job.

### States
Workflows may appear with one of the following states:

- RUNNING: Workflow is in progress
- NOT RUN: Workflow was never started
- CANCELLED: Workflow was cancelled before it finished
- FAILING: A job in the workflow has failed. Workflows go into Failing state when one of the jobs within the graph has failed while other jobs are still running. Failing state indicates that the workflow is eventually going to fail.
- FAILED: One or more jobs in the workflow failed. Failed state is when one or more jobs in the workflow graph have failed. Failed is a terminal state.
- SUCCESS: All jobs in the workflow completed successfully
- ON HOLD: A job in the workflow is waiting for approval
- NEEDS SETUP: A workflow stanza is not included or is incorrect in the config.yml file for this project


### Limitations
Projects that have pipelines enabled may use the CircleCI API to trigger workflows. Projects that do not enable pipelines will run as if the workflows did not exist when triggered by the API. Note: Builds without workflows require a ```build``` job.

Refer to the [Workflows](https://circleci.com/docs/2.0/faq/) section of the FAQ for additional information and limitations.

### Workflows Configuration Examples
For a full specification of the ```workflows``` key, see the [Workflows](https://circleci.com/docs/2.0/configuration-reference/#workflows) section of the Configuring CircleCI document.

**Note:** Projects configured with Workflows often include multiple jobs that share syntax for Docker images, environment variables, or ```run``` steps. Refer the [YAML Anchors/Aliases](https://yaml.org/spec/1.2/spec.html#id2765878) documentation for information about how to alias and reuse syntax to keep your ```.circleci/config.yml``` file small. See the [Reuse YAML in the CircleCI Config](https://circleci.com/blog/circleci-hacks-reuse-yaml-in-your-circleci-config-with-yaml/) blog post for a summary.

To run a set of parallel jobs, add a new ```workflows```: section to the end of your existing ```.circleci/config.yml``` file with the version and a unique name for the workflow. The following sample ```.circleci/config.yml``` file shows the default workflow orchestration with two parallel jobs. It is defined by using the ```workflows```: key named ```build_and_test``` and by nesting the ```jobs:``` key with a list of job names. The jobs have no dependencies defined, therefore they will run in parallel.

```yml
jobs:
  build:
    docker:
      - image: circleci/<language>:<version TAG>
    steps:
      - checkout
      - run: <command>
  test:
    docker:
      - image: circleci/<language>:<version TAG>
    steps:
      - checkout
      - run: <command>
workflows:
  version: 2
  build_and_test:
    jobs:
      - build
      - test
```
Parallel Workflow config example: 

```yml
jobs:
  lintering:
    steps:
      # lintering steps in one container
      
  run_specs:
    parallelism: 2
    steps:
      # test steps in parallel containers

workflows:
  version: 2
  build:
    jobs:
      - lintering
      - run_specs:
          requires:
            - lintering

```
## Docker Layer Caching (DLC)
**`$` Premium Feature Notice: Docker Layer Caching**

You can use Docker Layer Caching on CircleCI 2.0 for an additional fee.

Docker Layer Caching is a feature to use if building your own Docker images is a regular part of your CI/CD process. DLC will save image layers created within your jobs, rather than impact the actual container used to run your job. In short, the less your Dockerfiles change from commit to commit, the faster your image-building steps will run.

**Note:** in order to use DLC you need to switch to **Performance Plan** and for each build you will need to pay 200 additional credits

```yml
version: 2 
jobs: 
 build: 
   docker: 
     # DLC does nothing here, its caching depends on commonality of the image layers.
     - image: circleci/node:9.8.0-stretch-browsers 
   steps: 
     - checkout 
     - setup_remote_docker: 
         docker_layer_caching: true 
     # DLC will explicitly cache layers here and try to avoid rebuilding.
     - run: docker build .
```
### Examples

you can see how to use docker layer caching [here](https://youtu.be/AL7aBN7Olng)








