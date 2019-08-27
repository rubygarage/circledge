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

## Executors and Images

#### Overview

CircleCI enables you to run jobs in one of three environments:

* Within Docker images (docker)
* Within a Linux virtual machine (VM) image (machine)
* Within a macOS VM image (macos)
* Within a Windows VM image

It is possible to specify a different executor type for every job in your `.circleci/config.yml` by specifying the 
executor type and an appropriate image.
Main differences between using Docker and VM:

| Virtual Environment                        | Docker  | Machine   |
|--------------------------------------------|---------|-----------|
| Start time                                 | Instant | 30-60 sec |
| Clean environment                          | Yes     | Yes       |
| Custom images                              | Yes     | No        |
| Build Docker images                        | Yes     | Yes       |
| Full control over job environment          | No      | Yes       |
| Full root access                           | No      | Yes       |
| Run multiple databases                     | Yes     | Yes       |
| Run multiple versions of the same software | No      | Yes       |
| Layer caching                              | Yes     | Yes       |
| Run privileged containers                  | No      | Yes       |
| Use docker compose with volumes            | No      | Yes       |
| Configurable resources (CPU/RAM)           | Yes     | No        |

The docker key defines Docker as the underlying technology to run your jobs using Docker Containers. 
Containers are an instance of the Docker Image you specify. The first image listed in your configuration is 
the primary container image in which all steps run. All other containers run in a common network and every exposed 
port will be available on localhost from a primary container.

#### Pre-Built CircleCI Docker Images

For convenience, CircleCI maintains several Docker images. These images are typically extensions of official Docker 
images and include tools especially useful for CI/CD. All of these pre-built images are available in the [CircleCI org
on Docker Hub](https://hub.docker.com/search?q=circleci&type=image). 

CircleCI’s convenience images fall into two categories: **language images** and **service images**. 
All images add a circleci user as a system user.

#### Language Images

Language images are convenience images for common programming languages. These images include both the relevant 
language and commonly-used tools. A language image should be listed first under the docker key in your configuration, 
making it the primary container during execution. Only one Language image could be listed.

CircleCI maintains images for the languages below.

* Android
* Clojure
* Elixir
* Go (Golang)
* JRuby
* Node.js
* OpenJDK (Java)
* PHP
* Python
* Ruby
* Rust

#### Language Image Variants

CircleCI maintains several variants for language images. To use these variants, add one of the following suffixes 
to the end of an image tag.

* `-node` includes Node.js for polyglot applications
* `-browsers` includes Chrome, Firefox, Java 8, and Geckodriver
* `-browsers-legacy` includes Chrome, Firefox, Java 8, and PhantomJS
* `-node-browsers` combines the -node and -browsers variants
* `-node-browsers-legacy` combines the -node and -browsers-legacy variants

#### Service Images

Service images are convenience images for services like databases. These images should be listed after language images 
so they become secondary service containers.

CircleCI maintains images for the services below.

* buildpack-deps
* DynamoDB
* MariaDB
* MongoDB
* MySQL
* PostgreSQL
* Redis

#### Using Custom-Built Docker Images
If the CircleCI convenience images do not suit your needs, consider creating a custom Docker image for your jobs. 

There are two major benefits of doing this:
* **Faster job execution** – Packaging your required tools into a custom image removes the need to install
 them for every job.

* **Cleaner configuration** – Adding lengthy installation scripts to a custom image reduces the number of lines 
in your `config.yml` file

How to create docker image you can see in [CircleCi documentation](https://circleci.com/docs/2.0/custom-images/).

#### Best Practices
Convenience images are based on the most recently built versions of upstream images, so it is best practice to use 
the most specific image possible. This makes your builds more deterministic by preventing an upstream image from 
introducing unintended changes to your image.

CircleCI bases pre-built images off of upstream, for example, `circleci/ruby:2.4-node` is based off the most up to date 
version of the Ruby 2.4-node container. Using `circleci/ruby:2.4-node` is similar to using `:latest`. It is best
practice to lock down aspects of your build container by specifying an additional tag to pin down the image in 
your configuration.

That is, to prevent unintended changes that come from upstream, instead of using `circleci/ruby:2.4-node` use a more 
specific version of these containers to ensure the image does not change with upstream changes until you change 
the tag.

It is also possible to specify all the way down to the specific SHA of the image you want to use. Doing so allows 
you to test specific images for as long as you like before making any changes.

There are two ways to make an image more specific:

* Use a tag to pin an image to a version or operating system (OS).
* Use a Docker image ID to pin an image to a fixed version.

#### Sample Configuration

```yml
version: 2
jobs:
  build:
    docker:
      - image: circleci/ruby:2.5.1-node # primary image
      # service images
      - image: circleci/postgres:9.6
        environment: # environment variables for image
          POSTGRES_USER: postgres
          POSTGRES_DB: rails
          POSTGRES_PASSWORD: password
      - image: redis
```

## Environment variables
## Caching
## Containers/Parallelism/Jobs
## Project configurations
## Steps	
## Jobs
## Workflows
