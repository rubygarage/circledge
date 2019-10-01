# Orbs + Sample of Orb

## Orbs
**CircleCI Orbs** are shareable packages of configuration elements, including jobs, commands, and executors. CircleCI provides certified orbs, along with 3rd-party orbs authored by CircleCI partners. It is best practice to first evaluate whether any of these existing orbs will help you in your configuration workflow. Refer to the [CircleCI Orbs Registry](https://circleci.com/orbs/registry/) for the complete list of certified orbs.

## Using CircleCI Orbs

If you have chosen to use CircleCI orbs in your workflows and jobs, there are several different ways that you use orbs. You may choose to either import an existing orb (CircleCI and partner-certified orbs) from the CircleCI Orb Registry, or author your own orb for your specific workflow. Each of these approaches is described below.

### Importing an Existing Orb
To import an existing orb, perform the steps listed below.

1) Add a single line to to your version 2.1 `.circleci/config.yml` file for each orb, for example:
```yml
version: 2.1
```
**Note:** If your project was added to CircleCI prior to 2.1, you must enable “pipelines” to use the orbs key.

2) Add the `orbs` stanza below your version, invoking the orb. For example:
```yml
orbs:
  slack: circleci/slack@0.1.0
  heroku: circleci/heroku@0.0.1
```

### Importing a Partner Orb
To import a partner orb, perform the steps listed below.

1) Import the `orbs` key in your `.circleci.yml/config.yml` file in your configuration.

2) Replace the `<orb reference string>` value in the example shown below with the orb you wish to import from the [CircleCI Orbs Registry](https://circleci.com/orbs/registry/). There are a large number of CircleCI-certified and Partner-certified orbs that you may choose from.
```yml
version: 2.1  

orbs:
  <orb reference string>
```

### Authoring Your Own Orb
If you find that there are no existing orbs that meet your needs, you may author your own orb to meet your specific environment or configuration requirements by using the [CircleCI CLI](https://circleci.com/docs/2.0/local-cli/#overview) as shown in the circleci orb help output below. 

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

### Namespaces
Namespaces are used to organize a set of orbs. Each namespace has a unique and immutable name within the registry, and each orb in a namespace has a unique name. For example, the circleci/rails orb may coexist in the registry with an orb called username/rails because they are in separate namespaces.

**Note:** Namespaces are owned by organizations. Organizations are, by default, limited to claiming only one namespace. 

### Semantic Versioning in Orbs
Orbs are published with the standard 3-number semantic versioning system:

- major
- minor
- patch

## Publishing Orb

![ci](images/orbs_block_schema.png)

### Step 1 - Set Up the CircleCI CLI
Although it is possible to CI/CD orb publishing using the `orbs-tool` orb, the most direct and iterable way to build, publish, and test orbs is by using our CLI. Follow the steps below to install and then configure the CircleCI CLI.

- [Install the CircleCI CLI.](https://circleci.com/docs/2.0/orb-author-cli/#install-the-cli-for-the-first-time)
- [Update the CLI.](https://circleci.com/docs/2.0/orb-author-cli/#update-the-cli)
- [Configure the CLI.](https://circleci.com/docs/2.0/orb-author-cli/#configure-the-circleci-cli)

### Step 2 - Verify You Installed the CLI Correctly
Once you have configured the CircleCI CLI, verify you installed the CLI correctly and the CLI is updated and configured properly before beginning to work with orbs

### Step 3 - Bump Version Property to Orbs-Compatible 2.1
After validating your build configuration, bump the version property to 2.1 so it is compatible for use with orbs.

### Step 4 - Create a New Orb Using Inline Template
Using inline orbs are the easiest way to get started with orbs because you can reference them from your existing configuration. Although not required for orb authoring, using inline orbs can simplify the process and is a reasonable approach to authoring orbs quickly and easily

### Step 5 - Design Your Orb
Depending on whether you use an inline template or author your orb independent of this inline template, you will want to add elements (Jobs, Commands, and Executors) to your orb.

### Step 6 - Validate your Orb
When you have finished authoring your orb, simply run the `validate` command from your CLI. CircleCI provides several different tools to validate your orb, including the `circleci/orb-tools` orb.

### Step 7 - Publish Your Orb
The final step in the orb publishing process is for you to simply publish your orb using the `orb-tools/publish` CLI command in the `circleci/orb-tools` orb. Note that dev orb versions make it possible to publish multiple versions of an orb name (dev orbs are mutable).




