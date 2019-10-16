# Special Cases + examples

## Coverage per several threads
Before testing, make sure that your config is configured **correctly**

Simplcov **must** be enabled at the **beginning** in the rail helper

`spec/rails_helper.rb`:
```ruby
require 'simplecov'

SimpleCov.start do
# add filters here if you need
end

ENV['RAILS_ENV'] ||= 'test'
require File.expand_path('../config/environment', __dir__)
#...
```
`rails_helper` **must** be required in `.rspec`

`.rspec`:
```ruby
--require rails_helper
```


#### Code
`.circleci/config.yml`:
```yml
version: 2.1

caches: 
  - &bundle_cache v1-repo-{{ checksum "Gemfile.lock"  }}

executors:
  default:
    working_directory: ~/repo
    description: The official CircleCI Ruby Docker image
    docker:
      - image: circleci/ruby:2.6.1-node-browsers
        environment:
          RAILS_ENV: test
      - image: circleci/postgres:11.3-alpine
        environment:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: 'test'
          POSTGRES_DB: test
          DATABASE_URL: "postgres://ubuntu@localhost:5432/test"

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

  defaults:
    steps:
      - checkout
      - restore_cache:
          key: *bundle_cache
      - run: bundle install --path vendor/bundle
      - save_cache:
          key: *bundle_cache
          paths:
            - ~/repo/vendor/bundle
      - run:
          name: Set up DB
          command: |
            bundle exec rake db:create
            bundle exec rake db:migrate

jobs:
  lintering:
    executor: default
    steps:
      - defaults
      - run_linters
      
  run_specs:
    executor: default
    parallelism: 2
    steps:
      - defaults
      - run_specs

  check_coverage:
    executor: default
    steps:
      - defaults
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
require 'simplecov'
require 'json'
class SimpleCovHelper
  def self.report_coverage(base_dir: './coverage_results') # rubocop:disable Metrics/MethodLength
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

`lib/tasks/simplecov.rake`:

```ruby
require_relative '../../spec/support/helpers/simplecov_helper'
namespace :simplecov do
  desc 'merge_results'
  task report_coverage: :environment do
    SimpleCovHelper.report_coverage
  end
end
```
