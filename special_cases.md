
# Special Cases + examples

## 5. Lintering

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

It can be useful to choose `pronto` when you have old huge project, because `pronto` can lint newly added code ignoring old one. Thus you can handle such project lintering with `pronto`.

