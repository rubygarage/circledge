# Orbs + Sample of Orb

## Orbs
**CircleCI Orbs** are shareable packages of configuration elements, including jobs, commands, and executors. CircleCI provides certified orbs, along with 3rd-party orbs authored by CircleCI partners. It is best practice to first evaluate whether any of these existing orbs will help you in your configuration workflow. Refer to the [CircleCI Orbs Registry](https://circleci.com/orbs/registry/) for the complete list of certified orbs.

## Using CircleCI Orbs

If you have chosen to use CircleCI orbs in your workflows and jobs, there are several different ways that you use orbs. You may choose to either import an existing orb (CircleCI and partner-certified orbs) from the CircleCI Orb Registry, or author your own orb for your specific workflow. Each of these approaches is described below.

## Design Methodology
Before using orbs, you may find it helpful to understand the various design decisions and methodologies that were used when these orbs were designed. Orbs were designed with the following considerations:

- Orbs are transparent - If you can execute an orb, you and anyone else can view the source of that orb.

- Metadata is available - Every key can include a description key and an orb may include a description at the top level.

- Production orbs are always semantic versioned (semver’d) - CircleCI allows development orbs that have versions that start with dev:.

- Production orbs are immutable - Once an orb has been published to a semantic version, the orb cannot be changed. This prevents unexpected breakage or changing behaviors in core orchestration

- One registry (per install) - Each installation of CircleCI, including circleci.com, has only one registry where orbs can be kept.

## Orb Structure

Orbs consist of the following elements:

- Commands
- Jobs
- Executors

### Commands
Commands are reusable sets of steps that you can invoke with specific parameters within an existing job. For example, if you want to invoke the command `sayhello`, you would pass the parameter `to` as follows:

```yml
version: 2.1
jobs:
  myjob:
    docker:
      - image: "circleci/node:9.6.1"
    steps:
      - myorb/sayhello:
          to: "Lev"
```

### Jobs
Jobs are comprised of two parts: a set of steps, and the environment they should be executed within. Jobs are defined in your build configuration or in an orb and enable you to define a job name in a map under the jobs key in a configuration, or in an external orb’s configuration.

You must invoke jobs in the workflow stanza of config.yml file, making sure to pass any necessary parameters as subkeys to the job.

### Executors

When you declare an executor in a configuration outside of jobs, you can use these declarations for all jobs in the scope of that declaration, enabling you to reuse a single executor definition across multiple jobs.

An executor definition has the following keys available (some of which are also available when using the job declaration):

- docker, machine, windows, or macos
- environment
- working_directory
- shell
- resource_class

The example below shows a simple example of using an executor:

```yml
version: 2.1
executors:
  my-executor:
    docker:
      - image: circleci/ruby:2.4.0

jobs:
  my-job:
    executor: my-executor
    steps:
      - run: echo outside the executor
```



