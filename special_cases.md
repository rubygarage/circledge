# Special Cases + examples

## Lintering

There is an opportunity to control the quality of your code with a help of CircleCI using linters. You can configure CircleCI with one or several types of lintering:
- ### with linter aggregator (<a href="#overcommit">overcommit</a>, inquisition, etc.)
- ### with aggregator for Github mentions (<a href="#pronto">pronto</a>)
- ### with run of each linter
  You can also run each linter individually. It may be done with simple `run` step, for example:
  ```
  - run:
      name: Run rubocop
      command: bundle exec rubocop
  ```

  While it is simple to use each linter one by one, there are some drawbacks. At-first, you can't manage linters output on CI. At-second, some linters doesn't raise an exception, so step can be marked as 'success'. At-third, you can't manage which code to lint, the linter will proceed all project. And lastly, the first linter in a chain that returns an error will stop the execution of the job.

  But you can place `.sh` files with some script in `bin` directory, and then execute latter during CI build running:
  ```
  - run: bash bin/rubocop.sh
  ```
  `rubocop.sh` file example:
  #### Collect differences between your brach and base branch
  ```
  #!/usr/bin/env bash

  BASE_REMOTE=origin
  BASE_BRANCH=develop

  git fetch $BASE_REMOTE $BASE_BRANCH

  differences_list=()
  commit_list=`git --no-pager log --no-merges $BASE_REMOTE/$BASE_BRANCH...$CIRCLE_BRANCH | grep -e '^commit' | sed -e "s/^commit \(.\{8\}\).*/\1/"`
  changed_files_list = `git --no-pager diff $BASE_REMOTE/$BASE_BRANCH...$CIRCLE_BRANCH --name-only`

  for file in $changed_files_list; do
    for commit in $commit_list; do
      differences=`git --no-pager blame --follow --show-name -s $file | grep $commit | sed -e "s/^[^ ]* *\([^ ]*\) *\([0-9]*\)*).*$/\1:\2/"`
      for line in $differences; do
        differences_list+=("$line")
      done
    done
  done
  ```
  #### Collect errors
  ```
  error_counts=0
  error_lines=()

  while read line; do
    for difference in ${differences_list[@]}; do
      if [[ "$line" =~ "$difference" ]]; then
        error_counts=$(($error_counts + 1))
        error_lines+=("$line")
        break
      fi
    done
  done < <(bundle exec rubocop --format emacs)
  ```
  #### Exit with error message if error counts greater then zero
  ```
  if [ $error_counts -ne 0 ]; then
    echo -e "\033[31m$error_counts Lint Errors\033[0;39m"
    echo -e "\033[31m-----------------------------------\033[0;39m\n"
    printf '\033[31m%s\n' "${error_lines[@]}"
    echo -e "\033[31mPlease resolve them before merging.\033[0;39m"
    exit 1
  fi
  ```

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

    - as a minimal set of linters we advise to add also:
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
