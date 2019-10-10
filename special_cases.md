# Special Cases + examples

## Heroku continuous deployment

1) Create a Heroku account and follow the [Getting Started on Heroku](https://devcenter.heroku.com/start) documentation to set up a project in your chosen language

2) Add the name of your Heroku application (default: `HEROKU_APP_NAME`) and your Heroku API key (default: `HEROKU_API_KEY`) as environment variables. 

3) In your `.circleci/config.yml`, create a deploy job and add an executor type.

Example of config to setup Heroku deploy:
```yml
version: 2.1
orbs:
  heroku: circleci/heroku@1.0.0
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

More information about Heroku orb see [here](https://circleci.com/orbs/registry/orb/circleci/heroku)