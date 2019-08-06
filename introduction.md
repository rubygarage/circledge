# Introduction to Continuous Integration/Deployment/Delivery

## Description

## CircleCi - Analogues

|                              | [Circle CI](https://circleci.com/)   | [Travis CI](https://travis-ci.org/) | [Bamboo](https://www.atlassian.com/software/bamboo) | [GitLab CI](https://docs.gitlab.com/ee/ci/)       | [Jenkins](https://jenkins.io/) | [TeamCity](https://www.jetbrains.com/teamcity/) | [Codeship](https://codeship.com/) |
|------------------------------|--------------------------------------|-------------------------------------|-----------------------------------------------------|---------------------------------------------------|--------------------------------|-------------------------------------------------|-----------------------------------|
| Supports CD                  | yes                                  | no                                  | yes                                                 | yes                                               | yes                            | yes                                             | yes                               |
| Cloud hosting                | yes                                  | yes                                 | yes                                                 | yes                                               | yes                            | no                                              | yes                               |
| License                      | Proprietary                          | MIT                                 | Proprietary                                         | MIT/EE                                            | MIT                            | Proprietary                                     | Proprietary                       |
| Paid version price           | $50-3,150 per month                  | $69-489 per month                   | $10-110,000 one-off payment                         | $4-99 per month                                   | -                              | $299-21,999 one-off payment                     | $75-1,500 per month               |
| Free version                 | yes                                  | yes                                 | yes                                                 | yes                                               | yes                            | yes                                             | yes                               |
| Special feature              | Easy to use                          | Direct connectivity with GitHub     |                                                     | Awesome Docker support | A lot of plugins               | Gated commits                                   | Pro & Basic versions              |
| Docs and support             | Good                                 | Poor                                | Good                                                | Good                                              | Adequate                       | Good                                            | Poor                              |
| Learning curve and usability | Easy                                 | Easy                                | Medium                                              | Easy                                              | Easy                           | Medium                                          | Easy                              |
| Use case                     | For fast development and high budget | For small projects and startups     | For Atlassian integrations                          | For any project                                   | For big projects               | For enterprise needs                            | For any project                   |

## Pros and Cons

#### Main CircleCI strength

**Simple UI.** CircleCI is recognized for its user-friendly interface for managing builds/jobs. Its single-page web app is clean and easy to understand
**High-quality customer support.** StackShare’s community members highlight CircleCI’s speedy support: They respond to requests within 12 hours.
CircleCI runs **all types of software tests** including web, mobile, and container environments.
**Caching of requirements installation and third-party dependencies.** Instead of installing the environments, CircleCI can take data from multiple projects using the granular check-out-key options.
**No need for manual debugging.** CircleCI has debugging feature Debug via SSH, but Jenkins users have to debug manually by clicking on jobs.

## Images(System + Services)
## Environment variables
## Caching

Caching is one of the most effective ways to make jobs faster on CircleCI by reusing the data from expensive fetch operations from previous jobs.

A good example is package dependency managers such as Yarn, Bundler, or Pip. With dependencies restored from a cache, commands like yarn install will only need to download new dependencies, if any, and not redownload everything on every build.

#### Keys and Templates

A cache stores a hierarchy of files under a key.
Following are keys examples for different goals:
`myapp-{{ checksum "package-lock.json" }}` - Cache will be regenerated every time something is changed in package-lock.json file, different branches of this project will generate the same cache key.
`myapp-{{ .Branch }}-{{ checksum "package-lock.json" }}` - Cache will be regenerated every time something is changed in package-lock.json file, different branches of this project will generate separate cache keys.
`myapp-{{ epoch }}` - Every build will generate separate cache keys.
During step execution, the templates above will be replaced by runtime values and use the resultant string as the key.

The following table describes the available cache key templates:

| **Template**                        | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|---------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| {{ .Branch }}                   | The VCS branch currently being built.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| {{ .BuildNum }}                 | The CircleCI job number for this build.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| {{ .Revision }}                 | The VCS revision currently being built.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| {{ .Environment.variableName }} | The environment variable variableName (supports any environment variable exported by CircleCI or added to a specific Context—not any arbitrary environment variable).                                                                                                                                                                                                                                                                                                      |
| {{ checksum "filename" }}       | A base64 encoded SHA256 hash of the given filename’s contents, so that a new cache key is generated if the file changes. This should be a file committed in your repo. Consider using dependency manifests, such as package-lock.json, pom.xml or project.clj. The important factor is that the file does not change between restore_cache and save_cache, otherwise the cache will be saved under a cache key that is different from the file used at restore_cache time. |
| {{ epoch }}                     | The number of seconds that have elapsed since 00:00:00 Coordinated Universal Time (UTC), also known as POSIX or Unix epoch.                                                                                                                                                                                                                                                                                                                                                |
| {{ arch }}                      | Captures OS and CPU (architecture, family, model) information. Useful when caching compiled binaries that depend on OS and CPU architecture, for example, darwin-amd64-6_58 versus linux-amd64-6_62.                                                                                                                                                                                                                                      |

#### Full Example of Saving and Restoring Cache

```
steps:
  - checkout

  # Run bundler
  # Load installed gems from cache if possible, bundle install then save cache
  # Multiple caches are used to increase the chance of a cache hit

  - restore_cache:
      keys:
        # CircleCI restores caches in the order of keys listed in the restore_cache step.
        # Each cache key is namespaced to the project, and retrieval is prefix-matched.
        # The cache will be restored from the first matching key.
        - gem-cache-v1-{{ arch }}-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
        - gem-cache-v1-{{ arch }}-{{ .Branch }}
        - gem-cache-v1

  - run: bundle install --path vendor/bundle

  - save_cache:
      key: gem-cache-v1-{{ arch }}-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
      paths:
        - vendor/bundle

  # Precompile assets
  # Load assets from cache if possible, precompile assets then save cache
  # Multiple caches are used to increase the chance of a cache hit

  - restore_cache:
      keys:
        - asset-cache-v1-{{ arch }}-{{ .Branch }}-{{ .Environment.CIRCLE_SHA1 }}
        - asset-cache-v1-{{ arch }}-{{ .Branch }}
        - asset-cache-v1

  - run: bundle exec rake assets:precompile

  - save_cache:
      key: asset-cache-v1-{{ arch }}-{{ .Branch }}-{{ .Environment.CIRCLE_SHA1 }}
      paths:
        - public/assets
        - tmp/cache/assets/sprockets

  - run: bundle exec rspec
```

#### Partial Dependency Caching Strategies
```
steps:
  - restore_cache:
      keys:
        - gem-cache-{{ arch }}-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
        - gem-cache-{{ arch }}-{{ .Branch }}
        - gem-cache
```
In the above example, if a dependency tree is partially restored by the second or third cache keys, Bundler will incorrectly install on top of the outdated dependency tree.
Instead of a cascading fallback, a more stable option is a single version-prefixed cache key.
```
steps:
  - restore_cache:
      keys:
        - v1-gem-cache-{{ arch }}-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
```

## Containers/Parallelism/Jobs
## Project configurations
## Steps	
## Jobs
## Workflows
