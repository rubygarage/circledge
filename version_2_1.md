# Features available in version 2.1

## Orbs
  CircleCI orbs are shareable packages of configuration elements, including jobs, commands, and executors.
  more information about orbs you can see [here](orbs.md)

  The following example calls an Orb named hello-build that exists in the certified circleci namespace.

  ```yml
  version: 2.1
  orbs:
    hello: circleci/hello-build@0.0.5
  workflows:
    "Hello Workflow":
      jobs:
        - hello/hello-build
```

## Executors

Executors define the environment in which the steps of a job will be run, allowing you to reuse a single executor definition across multiple jobs.

```yml
version: 2.1
executors:
  my-executor:
    docker:
      - image: circleci/ruby:2.5.1-node-browsers

jobs:
  my-job:
    executor: my-executor
    steps:
      - run: echo outside the executor
```
See the [Using Parameters in Executors](https://circleci.com/docs/2.0/reusing-config/#using-parameters-in-executors) section of the [Reusing Config](https://circleci.com/docs/2.0/reusing-config/) document for examples of parameterized executors.

## `when` Step

A conditional step consists of a step with the key `when` or `unless`. Under the `when` key are the subkeys `condition` and `steps`. The purpose of the `when` step is customizing commands and job configuration to run on custom conditions (determined at config-compile time) that are checked before a workflow runs. See the [Conditional Steps section of the Reusing Config document](https://circleci.com/docs/2.0/reusing-config/#defining-conditional-steps) for more details.

Example:
```yml
version: 2.1

jobs: # conditional steps may also be defined in `commands:`
  job_with_optional_custom_checkout:
    parameters:
      custom_checkout:
        type: string
        default: ""
    machine: true
    steps:
      - when:
          condition: <<parameters.custom_checkout>>
          steps:
            - run: echo "my custom checkout"
      - unless:
          condition: <<parameters.custom_checkout>>
          steps:
            - checkout
workflows:
  build-test-deploy:
    jobs:
      - job_with_optional_custom_checkout:
          custom_checkout: "any non-empty string is truthy"
      - job_with_optional_custom_checkout
```

## Commands

A command definition defines a sequence of steps as a map to be executed in a job, enabling you to [reuse a single command definition](https://circleci.com/docs/2.0/reusing-config/) across multiple jobs.

Example: 
```yml
commands:
  sayhello:
    description: "A very simple command for demonstration purposes"
    parameters:
      to:
        type: string
        default: "Hello World"
    steps:
      - run: echo << parameters.to >>
```

# How to update 2.0 to 2.1

## Version 

Don't forget change version

```yml
versoin: 2.1
```

## Steps

Move your steps to commands if you use YAML anchors of define in the job.

Use:
```yml
commands:
  bundle_cache:
    steps:
      - restore_cache:
          keys:
           - repo-bundle-v2-{{ checksum ".ruby-version" }}-{{ checksum "Gemfile.lock" }}
      - run: bundle install --path vendor/bundle
      - save_cache:
          key: repo-bundle-v2-{{ checksum ".ruby-version" }}-{{ checksum "Gemfile.lock" }}
          paths:
            - ~/repo/vendor/bundle
jobs:
  build:
    steps:
      - bundle_cache

```
Instead:
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

jobs:
  build:
    steps:
      - <<: *restore_bundle_cache
      - <<: *bundle_install
      - <<: *save_bundle_cache
```

## Use executors instead anchors

Use:
```yml
executors:
  default:
    working_directory: ~/repo
    description: The official CircleCI Ruby Docker image
    docker:
      - image: circleci/ruby:2.6.1-node-browsers
```

Instead:
```yml
default: &default
  working_directory: ~/repo
  description: The official CircleCI Ruby Docker image
  docker:
    - image: circleci/ruby:2.6.1-node-browsers
```