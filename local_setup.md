# Local setup

Using the CircleCI Local CLI you can do:

- Debug and validate your CI config
- Run jobs locally
- Query CircleCI’s API
- Create, publish, view and manage Orbs

## Installation

Linux:
```
curl -fLSs https://circle.ci/cli | bash
```

Mac:
```
brew install circleci
```

Snap package (include CircleCI CLI, Docker):

```
sudo snap install docker circleci
sudo snap connect circleci:docker docker
```

## Updating

Update to the newest version:
```
circleci update
```

Check for updates manually (and not install them):
```
circleci update check
```

## Validate

To validate your config, navigate to a directory with a .circleci/config.yml file and run:
```
circleci config validate
```

If you are working with Orbs you can also validate your orb:
```
circleci orb validate /tmp/my_orb.yml
```

## Run A Job In A Container On Your Machine

You will need to have Docker installed on your system.
```
circleci local execute --job JOB_NAME
```

## Limitations of Running Jobs Locally

The CLI tool does not support:
- ```machine``` executor
- ```workflows```
- ```save_cache```
- ```restore_cache```
- ```store_artifacts```
- Environment Variables

## Uninstallation

Linux:

Remove the ```circleci``` executable from ```usr/local/bin```

Mac:
```
brew uninstall circleci
```

Snap package:
```
sudo snap remove circleci
```
