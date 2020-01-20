# Special Cases

## Coverage per several threads
When tests are run in several threads, there is a problem of incorrect display of coverage. To avoid this problem you can use the solution below.

`.circleci/config.yml`:
```yml
version: 2.1

executors:
...

commands:
  run_linters:
    steps:
    # run your linters here

      - run: mkdir coverage_results
      - persist_to_workspace:
          root: .
          paths:
            - coverage_results

  run_specs:
    steps:
      - run: 
          name: run specs
          environment:
            - RAILS_ENV: test
            - RACK_ENV: test
          command: |
            TEST_FILES="$(circleci tests glob "spec/**/*_spec.rb" | circleci tests split --split-by=timings)"
            bundle exec rspec --profile 10 \
                              --format progress \
                              --profile \
                              --fail-fast \
                              -- \
                              $TEST_FILES

      - attach_workspace:
          at: .      
      - run:
          name: Stash Coverage Results
          command: |
            cp -R coverage/.resultset.json coverage_results/.resultset-${CIRCLE_NODE_INDEX}.json
      - persist_to_workspace:
          root: .
          paths:
            - coverage_results

  merge_and_check_coverage:
    steps:
      - attach_workspace:
          at: .
      - run:
          name: Merge and check coverage
          command: |
            RUN_COVERAGE=true bundle exec rake simplecov:report_coverage
      - store_artifacts:
          path: ~/repo/coverage
          destination: coverage


jobs:
  lintering:
    executor: default
    steps:
      - run_linters
      
  run_specs:
    executor: default
    parallelism: 2
    steps:
      - run_specs

  check_coverage:
    executor: default
    steps:
      - merge_and_check_coverage

workflows:
  version: 2.1
  build:
    jobs:
      - lintering
      - run_specs:
          requires:
            - lintering
      - check_coverage:
          requires:
            - run_specs
```

`spec/support/helpers/simplecov_helper.rb`:

```ruby
class SimpleCovHelper
  def self.report_coverage(base_dir: './coverage_results')
    SimpleCov.start 'rails' do
      add_filter '/spec/'
      add_filter '/config/'
      add_filter '/vendor/'

      Dir['app/*'].each do |dir|
        add_group File.basename(dir).humanize, dir
      end

      minimum_coverage(90)
      merge_timeout(3600)
    end
    new(base_dir: base_dir).merge_results
  end

  attr_reader :base_dir

  def initialize(base_dir:)
    @base_dir = base_dir
  end

  def all_results
    Dir["#{base_dir}/.resultset*.json"]
  end

  def merge_results
    results = all_results.map { |file| SimpleCov::Result.from_hash(JSON.parse(File.read(file))) }
    SimpleCov::ResultMerger.merge_results(*results).tap do |result|
      SimpleCov::ResultMerger.store_result(result)
    end
  end
end
```

`lib/tasks/simplecov.rake`:

```ruby
if Rails.env.test?
  namespace :simplecov do
    desc 'merge_results'
    task report_coverage: :environment do
      require Rails.root.join('spec', 'support', 'helpers', 'simplecov_helper')
      SimpleCovHelper.report_coverage
    end
  end
end
```

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

## Approvable Deployment

Manual approvals can be used to enable controls and layers of risk reduction for jobs which require some level of monitoring. It is also the assurance that someone on your team can check and confirm the details of a job before deploying into production.

`workflows` can be configured to wait for manual approval of a job before continuing to the next job. **Anyone** who has push access to the repository can click the Approval button to continue the workflow. For example:
```yml
workflows:
  version: 2
  approvable-deployment:
    jobs:
      - hold:
          type: approval
      - deploy:
          requires:
            - hold
```

To use this feature you need:
- add the key `type: approval` to a job-predecessor, thus, job that should be manually approved must be after job with the `type: approval` key
- add `requires` key with name of the approval-job to all jobs that should run after approval
- to make approval move to CircleCI workflows section -> click `workflow` that "ON HOLD" -> click on job with "pause" sign -> click Approve

### Important information
- the key `type: approval` works **only** under the `workflow` key section
- the `hold` job must be a unique (arbitrary) name not used by any other job
- "on-hold" jobs do not have an expiration, and stay in such condition until approval or cancel
- unfortunately there is no way of triggering manual approval via CircleCI API (with `curl`) now


## Lintering

There is an opportunity to control the quality of your code with a help of CircleCI using linters. You can configure CircleCI with one or several types of lintering:
- #### with linter aggregator (<a href="#overcommit">overcommit</a>, inquisition, etc.)
- #### with aggregator for Github mentions (<a href="#pronto">pronto</a>)
- #### with run of each linter separately (for <a href="#rubocop">example</a>)

<div id="overcommit"></div>

### 5a. [overcommit](https://github.com/sds/overcommit)

`overcommit` is a flexible tool. In case you want to use `overcommit` you have to:
  - install gem `overcommit`
  - install every lintering gem that will be enabled in `overcommit` config file, such as `traceroute` etc.
  - create `overcommit.yml` file, the example of `overcommit` configuration, as well as scripts for `lol_dba` and `traceroute` you can find [here](https://gist.github.com/D3N/1318e9890c95142475b8c6f665283fb1).
  - create step for running `overcomit` in your CircleCI config file, for example:
  ```
  - run:
      name: Run overcommit
      command: |
        bundle exec overcommit -s
        bundle exec overcommit --sign pre-commit
        SKIP=AuthorEmail,AuthorName bundle exec overcommit -r
  ```

<div id="pronto"></div>

### 5b. [pronto](https://github.com/prontolabs/pronto)

It can be useful to choose `pronto` when you have big old project, because `pronto` can lint newly added code ignoring old one. Besides lintering, `pronto` has one interesting feature: it can leave comments with issue text produced by some linter on your Github pull requests.

  - To install `pronto` you have to:
    - add to your `Gemfile`:
      ```
      gem 'pronto'
      ```

    - as an optimal set of linters we advise to add also:
      ```
      gem 'pronto-brakeman', require: false
      gem 'pronto-bundler_audit', require: false
      gem 'pronto-fasterer', require: false
      gem 'pronto-flay', require: false
      gem 'pronto-rails_best_practices', require: false
      gem 'pronto-rails_schema', require: false
      gem 'pronto-reek', require: false
      gem 'pronto-rubocop', require: false
      ```

  - To use `pronto` locally run command:
    ```
    pronto run
    ```
    If your code has some issues, you will have console output from every `pronto-linter` added to `Gemfile`.

  - To use pronto with CircleCI and Github you have to:
    - generate Github personal access token:\
      go to your Github account user's `Settings -> Developer settings -> Personal access tokens -> Generete new token`

    - add this Github token to CircleCI project:\
      go to CircleCI project's `Settings -> Environment Variables -> Add Variable`,\
      Name - `PRONTO_GITHUB_ACCESS_TOKEN`\
      Value - insert your Github token

    - turn on "Only build pull requests":\
      go to CircleCI project's `Settings -> Advanced Settings -> Only build pull requests -> On`

    - add to your `.circleci/config.yml` such step:
      ```
      - run:
          name: Run lintering with pronto
          command: |
            PRONTO_PULL_REQUEST_ID=`echo $CIRCLE_PULL_REQUEST | grep -o -E "[0-9]+$"`
            bundle exec pronto run -f github_status github_pr -c origin/development || true
      ```
    In case of project's code issues, you should see pull request comments under each problematic string of code. Each comment will be equal to linter's issue text.

<div id="rubocop"></div>

### 5c. Using custom separate linter

You can also run each linter individually. It may be done with simple `run` step, for example:
  ```
  - run:
      name: Run rubocop
      command: bundle exec rubocop
  ```

  While it is simple to use each linter one by one, there are some drawbacks:
  - you can't manage linters output on CI
  - some linters doesn't raise an exception, so step can be marked as 'success'
  - you can't manage which code to lint, the linter will proceed all project
  - the first linter in a chain that returns an error will stop the execution of the job.

  But you can place `.sh` files with some script in `bin` directory, and then execute latter during CI build running, so you can handle all issues above:
  ```
  - run: bash bin/rubocop.sh
  ```
  [Here](https://gist.github.com/D3N/c66ff89e7cc9123a14f4e06dbdc87a9c) you can find `rubocop.sh` file example.

## Daily builds

It can be inefficient and expensive to run a workflow for every commit for every branch. Instead, you can schedule a workflow to run at a certain time for specific branches. This will disable commits from triggering jobs on those branches.

A `triggers` key should be added for indicating a scheduled build. Currently `triggers` have only one subkey - `schedule` which defines a certain time, for example a nightly build that runs every day at 00:00 :

```
workflows:
  version: 2
  commit:
    jobs:
      - test
      - deploy
  nightly:
    triggers:
      - schedule:
          cron: "0 0 * * *"
          filters:
            branches:
              only:
                - master
                - beta
    jobs:
      - coverage
```

`commit` and `nightly` are workflow names.

The `cron` key sets trigger time. It is defined using POSIX `crontab` syntax.

`filters` can have the key `branches`.

`branches` have subkeys `only` and `ignore` that give you an ability to choose which branch should have a schedule trigger created for it.

Example above executes builds when you push changes to a version control system and also every night on specified branches. You can use `nightly` workflow without any additional workflows, in this case there will be only 1 build execution per day.


## Manual Triggering

It can be useful when condionally running jobs or using it with github actions for example.

### `build` job triggering

With CircleCI API you can trigger `build` job manually. To make it you have to do several steps:

- go to user's Settings -> Personal API Tokens -> click Create New Token button

  ![ci](images/api_token.png)

- use generated token in `curl` command:
  ```
  curl -X POST --header “Content-Type:application/json” --data '"parallel": 2, {"build_parameters": {"FOO":"BAR"}}' https://circleci.com/api/v1.1/project/:vcs-type/:user-organization/:project-name/tree/:branch\?circle-token=:your-token
  ```

  `parallel` is an optional key which defines number of containers to use to run the build.

  To `build_parameters` (also optional) you can pass additional environment variables to inject into the build environment.

  In URI from example above replace with your values:

  `:vcs-type` - github or bitbucket

  `:user-organization` - user or organization name

  `:project-name` - project name

  `:branch` - branch to run build

  `:your-token` - API token created before

In case you have not `build` job in your config file, you will see an error.

### `workflows` triggering

You can trigger `workflows` with command:

```
curl -X POST https://circleci.com/api/v1.1/project/:vcs-type/:user-organization/:project-name/build?circle-token=:your-token
```

The command above will trigger workflows for default branch. You can also specify which branch to trigger with parameter `branch`:

```
curl -X POST --header “Content-Type:application/json” --data '{"branch":":your-branch"}' https://circleci.com/api/v1.1/project/:vcs-type/:user-organization/:project-name/build?circle-token=:your-token
```

## Auto-deploy to AWS

1) Create a AWS account and setup [EC2 instance](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)

2) Add `AWS_ACCESS_KEY_ID`, `AWS_DEFAULT_REGION`, `AWS_SECRET_ACCESS_KEY` to environment variables in project setting in circleci UI

3)