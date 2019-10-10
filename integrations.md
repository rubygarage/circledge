# Integrations 

**CircleCi** has the ability to integrate with different services to perform different tasks. 

for example: notifications(Slack), code storage(GitHub), cooperation(Jira), deployment(Heroku), etc.

Here we will look at the most used services for integrating with CircleCi

## Slack

**Before using** automatic notifications via slack, you need to **create and configure** the application in Slack. You can see how to do it [here](https://api.slack.com/incoming-webhooks).

sample of using slack's orb:

```yml
version: 2.1

orbs:
  slack: circleci/slack@x.y.z

jobs:
  build:
    docker:
      - image: <docker image>
    steps:
      - run: exit 0
      - slack/status:
          fail_only: true
          only_for_branches: only_for_branches
          color: '#42e2f4'
          message: This is a custom message notification
          webhook: <webhook>
```
**Note:** save your `webhook` in environment variables, otherwise you risk that it will be visible to everyone.

## Jira

Connect Jira with CircleCi

**Note:** You have to be an JIRA **admin** to install this plugin.

### How to install Jira 

1) Go to `Projects -> Settings -> JIRA integration`

2) Go to the Atlassian Marketplace to get the  [CircleCI JIRA Plugin](https://marketplace.atlassian.com/apps/1215946/circleci-for-jira?hosting=cloud&tab=overview)

3) Install the plugin and follow the prompts to set it up

4) Return to the CircleCI JIRA Integration settings page and add the generated token.

Click on the JIRA icon and select the following:

- Project name
- Issue type
- Issue summary
- Description

**Note:** The current JIRA plugin only supports default fields.

You’re all set to creating quick tickets from your job output page!

example of usage:

```yml
jobs:
  build:
    docker:
      - image: 'circleci/node:10'
    steps:
      - run: echo "hello"
orbs:
  jira: circleci/jira@1.0.5
version: 2.1
workflows:
  build:
    jobs:
      - build:
          post-steps:
            - jira/notify

```

## Cypress

Cypress is a front end automated testing tool. Cypress provides testing for anything that runs in a browser.

sample example of usage with yarn:

```yml
version: 2.1

orbs:
  cypress: cypress-io/cypress@1
workflows:
  build:
    jobs:
      - cypress/install:
          cache-key: >-
            cache-{{ arch }}-{{ .Branch }}-{{ checksum "frontend/package.json"
            }}
      - cypress/run:
          yarn: true
          cache-key: >-
            cache-{{ arch }}-{{ .Branch }}-{{ checksum "frontend/package.json"
            }}
          working_directory: frontend
```

More information about usage cypress orb see [here](https://circleci.com/orbs/registry/orb/cypress-io/cypress)


