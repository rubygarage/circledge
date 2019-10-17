# Integrations 

**CircleCi** has the ability to integrate with different services to perform different tasks. 

for example: notifications(Slack), code storage(GitHub), cooperation(Jira), deployment(Heroku), etc.

Here we will look at the most used services for integrating with CircleCi

## Slack

**Before using** automatic notifications via slack, you need to **create and configure** the application in Slack. You can see how to do it [here](https://api.slack.com/incoming-webhooks).

### Usage example:

#### Approval

if you need notifications to approve jobs:
```yml
version: 2.1

orbs:
  slack: circleci/slack@3.4.0

version: 2.1
workflows:
  your-workflow:
    jobs:
      - slack/approval-notification:
          message: Pending approval
          webhook: webhook
```
**Note:** save your `webhook` in environment variables, otherwise you risk that it will be visible to everyone.

default: ${SLACK_WEBHOOK}

#### Notify
Notify a slack channel with a custom message at any point in a job with this custom step. More docs [here](https://github.com/CircleCI-Public/slack-orb)

sample of using slack's orb:

```yml
version: 2.1

orbs:
  slack: circleci/slack@3.4.0
jobs:
  build:
    docker:
      - image: circleci/ruby:2.6.1-node-browsers
    steps:
      - slack/notify:
          channel: CHANNELID
          color: '#42e2f4'
          mentions: 'USERID1,USERID2,'
          message: This is a custom message notification
```

#### Status

Send a status alert at the end of a job based on success or failure. This **must** be the last step in a job.

```yml
version: 2.1

orbs:
  slack: circleci/slack@3.4.0
jobs:
  build:
    docker:
      - image: circleci/ruby:2.6.1-node-browsers
    steps:
      - slack/status:
          fail_only: true
          only_for_branches: develop
```

Parameters and other info about slack orb you can see [here](https://circleci.com/orbs/registry/orb/circleci/slack)

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

### Viewing Build and Deploy Statuses in Jira
With CircleCI orbs it is also possible to display your build and deploy status in Jira. To do this, you will need to:

- Make sure you followed the steps above to connect Jira with CircleCI.
- Make sure that you are using `version 2.1` at the top of your `.circleci/config.yml` file.
- If you do not already have pipelines enabled, go to **Project Settings -> Build Settings -> Advanced Settings** and enable them.
- To get an API token for build information retrieval, go to **Project Settings -> Permissions -> API Permissions** and create a token with Scope: all. Copy the token.
- To allow the integration to then use that key, go to **Project Settings -> Build Settings -> Environment Variables** and add a variable named CIRCLE_TOKEN with the value being the token you just made.
- Add the orb stanza, invoking the Jira orb.
- Use the Jira orb in a step.

example of usage:

```yml
version: 2.1

orbs:
  jira: circleci/jira@1.0.5
jobs:
  build:
    docker:
      - image: 'circleci/node:10'
    steps:
      - run: echo "hello"
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


