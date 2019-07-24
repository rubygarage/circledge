# Special Cases + examples

## 5. Lintering
For example we want to run on CI linter [lol_dba](https://github.com/plentz/lol_dba), current linter is not present in `pronto` or `overcommit` available gem list, so we need to setup it by-self.

1) You need to pay attention to the position of your linter step,
if we talk about `lol_dba` we need our project db ready, so put this step after `bundle install ` and `bundle exec rails db:migrate`

2) CI runs on test environment so you need to move your gem to test group:
```
group  :development, :test  do
  gem  'lol_dba', '~> 2.0'
end
```

2) Add to your `circleci/config.yaml` a new step, which will execute `lol_dba.sh` file.
```
steps:
  - run:
    name: run lol_dba
    command: bash bin/lol_dba.sh
```
3) Create a `lol_dba.sh` file in `bin` directory.

4) Fill in `lol_dba.sh` file with next content:
```
#!/usr/bin/env bash

OUTPUT=$(bundle exec lol_dba db:find_indexes)
FAILURE_FLAG = 'AddMissingIndexes'

# Exit with error message if output includes failure flag

if [[ $OUTPUT =~ $FAILURE_FLAG ]]; then
  echo -e "\e[31m-----------------------------------\e[0;39m\n"
  echo -e "\e[96mlol_dba: \033[31mFail\033[0;39m\n"
  while read -r line; do
    echo -e "\e[33m$line"
  done <<< "$OUTPUT"
  echo -e "\033[31m-----------------------------------\033[0;39m"
  exit 1
else
  echo -e "\e[96mlol_dba: \e[32mSuccess"
fi
```
So you need to specify a FAILURE_FLAG for your linter and handle it with error message and `exit 1` command.

If you want to setup some gem as `rubocop` which linter your code and is not present in  `pronto` or `overcommit` available gem list, you can do as in example below.
```
#!/usr/bin/env bash

# Collect differences between your brach and base branch

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

# Collect errors

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

# Exit with error message if error counts greater then zero

if [ $error_counts -ne 0 ]; then
  echo -e "\033[31m$error_counts Lint Errors\033[0;39m"
  echo -e "\033[31m-----------------------------------\033[0;39m\n"
  printf '\033[31m%s\n' "${error_lines[@]}"
  echo -e "\033[31mPlease resolve them before merging.\033[0;39m"
  exit 1
fi
```
