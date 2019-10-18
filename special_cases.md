# Special Cases

## Coverage per several threads
`.circleci/config.yml`:
```yml
version: 2.1

executors:
...

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


jobs:
  lintering:
    executor: default
    steps:
      - run_linters
      
  run_specs:
    executor: default
    parallelism: 2
    steps:
      - run_specs

  check_coverage:
    executor: default
    steps:
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
  def self.report_coverage(base_dir: './coverage_results')
    SimpleCov.start 'rails' do
      add_filter '/spec/'
      add_filter '/config/'
      add_filter '/vendor/'

      Dir['app/*'].each do |dir|
        add_group File.basename(dir).humanize, dir
      end

      minimum_coverage(90)
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
    results = all_results.map { |file| SimpleCov::Result.from_hash(JSON.parse(File.read(file))) }
    SimpleCov::ResultMerger.merge_results(*results).tap do |result|
      SimpleCov::ResultMerger.store_result(result)
    end
  end
end
```

`lib/tasks/simplecov.rake`:

```ruby
if Rails.env.test?
  namespace :simplecov do
    desc 'merge_results'
    task report_coverage: :environment do
      require Rails.root.join('spec', 'support', 'helpers', 'simplecov_helper')
      SimpleCovHelper.report_coverage
    end
  end
end
```
