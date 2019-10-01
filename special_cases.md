# Special Cases + examples

## Daily builds

### Scheduled Builds
With CircleCI you can schedule jobs and workflows, in case you need your job repeat or run automatically. To achieve such behaviour you can use `triggers` key that specifies which triggers will cause the workflow to be executed. Default behavior is to trigger the workflow when pushing to a branch.

A workflow may have a `schedule` indicating it runs at a certain time, for example a nightly build that runs every day at 00:00 :

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

The `cron` key sets trigger time. It is defined using POSIX `crontab` syntax.

`filters` can have the key `branches`.

`branches` have subkeys `only` and `ignore` that give you an ability to choose which branch should have a schedule trigger created for it.

Example above executes builds when you push changes to a version control system and also every night on specified branches. You can use `nightly` workflow only in case you need to reduce load on available containers.

### Manual Builds Triggering

