# Special Cases + examples

## Approvable Deployment

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
