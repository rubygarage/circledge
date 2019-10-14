# Special Cases + examples

## Heroku continuous deployment

1) Create a Heroku account and follow the [Getting Started on Heroku](https://devcenter.heroku.com/start) documentation to set up a project in your chosen language

2) Add the name of your Heroku application (default: `HEROKU_APP_NAME`) and your Heroku API key (default: `HEROKU_API_KEY`) as environment variables. 

    - `HEROKU_APP_NAME` - You can find and change the name of the application on the settings page on Heroku

    - `HEROKU_API_KEY` -  You can display the token via the CLI: `$ heroku auth:token`

3) In your `.circleci/config.yml`, create a deploy job and add an executor type.

Example of config to setup Heroku deploy:
```yml
version: 2.1
orbs:
  heroku: circleci/heroku@0.0.8
workflows:
  heroku_deploy:
    jobs:
      - deploy
jobs:
  deploy:
    executor: heroku/default 

    steps:
      - checkout
      - heroku/install
      - heroku/deploy-via-git: 
          only-branch: master
```

if you using config version 2

```yml
version: 2
jobs:
  build:
    ...
  deploy:
    docker:
      - image: buildpack-deps:trusty
    steps:
      - checkout
      - run:
          name: Deploy Master to Heroku
          command: |
            git push https://heroku:$HEROKU_API_KEY@git.heroku.com/$HEROKU_APP_NAME.git master

workflows:
  version: 2
  build-deploy:
    jobs:
      - build
      - deploy:
          requires:
            - build
          filters:
            branches:
              only: master
```

If you need to configure deploy from different branches to different instances. 

For example: development for staging and master for production. You can solve this problem by example below

```yml
version: 2.1

orbs:
  heroku: circleci/heroku@0.0.8

jobs:
  deploy_prod_heroku:
    executor: heroku/default 

    steps:
      - checkout
      - heroku/install
      - heroku/deploy-via-git: 
          only-branch: master
          app-name: ${HEROKU_APP_NAME_PROD}
          api-key: ${HEROKU_API_KEY_PROD}

  deploy_staging_heroku:
    executor: heroku/default 

    steps:
      - checkout
      - heroku/install
      - heroku/deploy-via-git: 
          only-branch: develop
          app-name: ${HEROKU_APP_NAME_STAGING}
          api-key: ${HEROKU_API_KEY_STAGING}

workflows:
  version: 2.1
  build:
    jobs:
      - build
      - deploy_prod_heroku:
          requires:
            - build
      - deploy_staging_heroku:
          requires:
            - build

```

More information about Heroku orb see [here](https://circleci.com/orbs/registry/orb/circleci/heroku)