# Orbs + Sample of Orb

## Orbs
**CircleCI Orbs** are shareable packages of configuration elements, including jobs, commands, and executors. CircleCI provides certified orbs, along with 3rd-party orbs authored by CircleCI partners. It is best practice to first evaluate whether any of these existing orbs will help you in your configuration workflow. Refer to the [CircleCI Orbs Registry](https://circleci.com/orbs/registry/) for the complete list of certified orbs.

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

## Creating a CircleCI Orb

![ci](images/orbs_block_schema.png)

### CircleCI Settings
In the CircleCI app Settings page for your project, pipelines must be enabled (default is to be ON for all new projects). The organization owner must also opt-in to use of uncertified orbs in your organization under the Settings tab on the Security page of the CircleCI app.

### Setup circleci CLI
The CircleCI platform enables you to write orbs using the CircleCI CLI. If you choose to work with the CLI, the process of writing orbs will be more efficient because you will be able to use existing CircleCI CLI tools and commands.

How to **setup** circleci CLI you can see [**here**](https://github.com/rubygarage/circledge/blob/develop/local_setup.md)

### Configure circleci CLI

run in your console: 
```
circleci setup
```
after running this command you need to set your API token and host (default "https://circleci.com")

**Note:** before running this command you need to **create** an API token in the UI

after setup host and API token you can to check your configurated CLI.
```
circleci diagnostic
```

output: 
```
---
CircleCI CLI Diagnostics
---
Debugger mode: false
Config found: /Users/<username>/.circleci/cli.yml
API host: https://circleci.com
API endpoint: graphql-unstable
OK, got a token.
Trying an introspection query on API...
Ok.
Hello, <your name>.
```

### Change settings for organization to use 3-rd part orbs

You need to go into the **UI settings** of your organization and allow the use of not certified orbs

### Create namespace 

Orbs are stored under specific namespaces. Before creating an orb, you need to **create a namespace** or use the existing one.

for create namespace run this command:

```
circleci namespace create <name> <VCS> <organization>
```

### Create and publish your own Orb

for creating orb run: 

```
circleci orb create <namespace>/<orb>
```

sample of orb:

```
version: 2.1
commands:
  say_hello:
    description: "A very simple command for demonstration purposes"
    parameters:
      to:
        type: string
        default: "Hello World"
    steps:
      - run: echo << parameters.to >>
```

after that you need to validate the orb you wrote. To do this, run the command:

```
circleci orb validate <path to your created orb> 
```

to use the orb you need to publish it on circleci. run this command

```
circleci orb publish <path> <namespace>/<orb>@<version>
```
