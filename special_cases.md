# Special Cases + examples

## Coverage per several threads
#### Enabling Formatters
Test metadata is not automatically collected in CircleCI 2.0 until you enable the JUnit formatters. For RSpec, and Minitest, add the following configuration to enable the formatters:
RSpec requires the following be added to your gemfile:
`gem 'rspec_junit_formatter'`
Minitest requires the following be added to your gemfile:
`gem 'minitest-ci'`

#### Metadata Collection in Custom Test Steps
Write the XML files to a subdirectory if you have a custom test step that produces JUnit XML output as is supported by most test runners in some form, for example:
```
- store_test_results:
    path: /tmp/test-results
```

#### Code
`.circleci/config.yml`:
```
defaults: &defaults
  working_directory: ~/project_name
  parallelism: 2
  docker:
    - image: circleci/ruby:2.6.3-node-browsers-legacy
      environment:
        BUNDLE_JOBS: 3
        BUNDLE_RETRY: 3
        BUNDLE_PATH: vendor/bundle
        PGHOST: 127.0.0.1
        PGUSER: circleci-demo-ruby
        RAILS_ENV: test
    - image: circleci/postgres:10.6-alpine-ram
      environment:
        POSTGRES_USER: circleci-demo-ruby
    - image: circleci/redis:4.0.13-alpine
version: 2
jobs:
  test:
    <<: *defaults
    steps:
      - checkout

      - save_cache:
          key: v2-repo-{{ .Environment.CIRCLE_SHA1 }}
          paths:
            - ~/products_marketplace_app

      - restore_cache:
          keys:
            - gem-cache-{{ arch }}-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
            - gem-cache-{{ arch }}-{{ .Branch }}
            - gem-cache
      - run:
          name: Install Bundler
          command: gem install bundler
      - run:
          name: Which bundler?
          command: bundle -v
      - run:
          name: Bundle Install
          command: bundle check || bundle install
      - save_cache:
          key: gem-cache-{{ arch }}-{{ .Branch }}-{{ checksum "Gemfile.lock" }}
          paths:
            - ~/project_name/vendor/bundle

      - run:
          name: Wait for DB
          command: dockerize -wait tcp://localhost:5432 -timeout 1m
      - run:
          name: Database setup
          command: bin/rails db:create db:schema:load --trace
      - run:
          name: Wait for Redis
          command: |
            dockerize -wait tcp://localhost:6379 -timeout 2m
      - run:
          name: Parallel Rspec
          environment:
            - RAILS_ENV: test
            - RACK_ENV: test
          command: |
            mkdir -p /tmp/rspec
            TEST_FILES="$(circleci tests glob "spec/**/*_spec.rb" | circleci tests split --split-by=timings)"
            bundle exec rspec --profile 10 \
                              --format RspecJunitFormatter \
                              --out /tmp/rspec/rspec.xml \
                              --format progress \
                              --profile \
                              --fail-fast \
                              -- \
                              $TEST_FILES
      - store_test_results:
          path: /tmp/rspec
      - run:
          name: Stash Coverage Results
          command: |
            mkdir coverage_results
            cp -R coverage/.resultset.json coverage_results/.resultset-${CIRCLE_NODE_INDEX}.json
      - persist_to_workspace:
          root: .
          paths:
            - coverage_results
  coverage:
    <<: *defaults
    steps:
      - attach_workspace:
          at: .
      - restore_cache:
          key: v2-repo-{{ .Environment.CIRCLE_SHA1 }}
      - run:
          name: Merge and check coverage
          command: |
            RUN_COVERAGE=true bundle exec rake simplecov:report_coverage
      - store_artifacts:
          path: ~/project_name/coverage
          destination: coverage

workflows:
  version: 2
  build_and_test:
    jobs:
      - test
      - coverage:
          requires:
            - test
```
`spec/rails_helper.rb`
```
# ...
if ENV['RAILS_ENV'] == 'test'
  require 'simplecov'
  SimpleCov.start 'rails'
end
# ...
```
`spec/support/helpers/simplecov_helper.rb`:
```
# frozen_string_literal: true

require 'simplecov'

class SimpleCovHelper
  def self.report_coverage(base_dir: './coverage_results')
    SimpleCov.start 'rails' do
      skip_check_coverage = ENV.fetch('SKIP_COVERAGE_CHECK') { false }

      minimum_coverage(90) unless skip_check_coverage
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
    results = all_results.map do |file|
      SimpleCov::Result.from_hash(JSON.parse(File.read(file)))
    end
    SimpleCov::ResultMerger.merge_results(*results).tap do |result|
      SimpleCov::ResultMerger.store_result(result)
    end
  end
end
```
`lib/tasks/simplecov_parallel.rake`:
```
# frozen_string_literal: true

if Rails.env.test?
  require_relative '../../spec/support/helpers/simplecov_helper'
  namespace :simplecov do
    desc 'merge_results'
    task report_coverage: :environment do
      SimpleCovHelper.report_coverage
    end
  end
end
```
