---
layout: post
title: Optimize your Ruby on Rails CI pipeline
live: false
excerpt: Continuous Integration is an amazing thing, it allows us to automatically run thousands of checks to ensure that our changes are production-ready, and it's doing it fast... well, at least it should be fast.
---

***
#### This article is heavily based on my Poznan Ruby User Group presentation, you can preview the presentation [there](#TODO).
***
Some time ago I've noticed that a very short pipeline - one that consists of only 300 tests takes about 5 minutes to finish. I've quickly checked some bigger(in terms of number of tests) projects I've contributed just to discover that 10 minutes is sometimes completely acceptable time?!

Then I checked if there are any statistics on the average build time - our friends from SemaphoreCI [seems to prove my observations](https://semaphoreci.com/blog/2018/04/11/state-of-testing-in-rails.html).

The discovery gave me only one idea - there must be a room for improvement. Quick Google around and I've found out that Steph Skardal already wrote two awesome posts in this topic:
- [9 Steps to an Optimized Ruby on Rails Monolith Pipeline](https://medium.com/upstart-tech/9-steps-to-an-optimized-ruby-on-rails-monolith-pipeline-acc160823cee)
- [Speed Up CI (Continuous Integration) 2022](https://medium.com/upstart-tech/speed-up-ci-continuous-integration-2022-384c93bff5d7)
But the topic is so wide that there is definitely more that could be done and I decided to document all of my findings from the few projects I've had the chance to optimize the pipeline.

## Why spend time on optimizing?
There are a few main points worth consideration:
- It increases your iteration speed - the faster you can iterate the more you can learn
- Faster iteration = shorter feedback loop
- Successful companies like Facebook puts emphasis on *Move fast and break things* mantra. Some of you might think that breaking things isn't the right thing to do, but in fact, learning through experiments is very effective. The gotcha is there that we want to discover the failure as soon as possible.
- For ones who prefer number arguments: According to [Kelly Sutton's blog](https://kellysutton.com/2020/05/18/speeding-up-a-rails-continuous-integration-pipeline.html) - Gusto lands 2% more Pull Requests per engineer per week for each minute of CI time improvement.

# Navigation
It's the longest post I've ever written and reading it as a whole might be difficult. I'd recommend using the anchor links from the next two sections that will send you straight into the info about subject/technology that might interest you the most.

I've tried to order the solutions from the ones that are the easiest to implement, to the more complex ones, however at some point it becomes impossible to distinguish the difficulty.

## What's slowing down the pipeline?
There are numerous reasons why it is slow, each case might be different but here I will try to address the most common scenarios that I met in my work:
- [Configuration problems](#check-your-configuration)
- Unefficient cache usage
- Feature/integration tests with Capybara
- Slow API requests to 3rd party services
- No/not enough parallelization

Possibly each of the points could deserve a deep dive in a separate article on its own. But my goal isn't to overload you with knowledge, I want to share solutions with you!

### Technologies that I worked on so far
As mentioned - every application might need a different strategy. I've worked only with limited amount of tools(tried to add anchor for any section related to the tool):
- RSpec - Optimize your test suite, Test-prof, Flaky Tests
- Github Actions - Caching, Use your own runner
- Capybara for Feature Tests -  Configuration
- Puffing Billy as a Request Proxy
- Parallel tests gem

# Solutions
## Check your configuration

1. Set **DB cleaner** settings to use **transaction strategy:** it's not possible for all projects to use this strategy, but is the fastest one refering to the [Database Cleaner gem](https://github.com/DatabaseCleaner/database_cleaner#what-strategy-is-fastest){:target="_blank"} homepage.
2. **Organize gems into Groups in Gemfile:** if you aren't leveraging Gemfile groups then every pipeline run you are loading some unnecessary Gems. Loading the Gems is usually pretty fast so it won't make a huge difference, but overall, **it's always a good idea to have a clean Gemfile.**
3. **Organize your _dependencies_ and _devDependencies_ in _package.json_:** the `node_modules` folder often grows to enormous sizes, so it's worth checking if you need everything from your package file.
4. **Disable logging in the CI environment or log only fatal messages:** if you aren't using `artifacts` to publish pipeline logs then it's a no-brainer to disable them or use only `fatal` log level:
```ruby
# config/environments/test.rb
config.log_level = :fatal
```

5. **Check Postgres health check settings:**
```yaml
# .github/workflows/{your_workflow.yml}
services:
  postgres:
	    image: postgres:11
	    ports: ["5432:5432"]
	    options: >-
	      --health-cmd pg_isready
	      --health-interval 2s
	      --health-timeout 2s
	      --health-retries 8
```
It's worth increasing the retries amount and lowering the interval time to not waste any seconds waiting for the next retry.
6. **Use `rails db:schema:load` instead of `rails db:migrate`:** running the migrations for a test database might be redundant, even though migrating an empty database is fast some of the projects are having thousands of them that will always take some time. If you have an up-to-date `schema.rb` file then I'd consider dumping it.

* * *
#### Did you know the differences between cache and artifacts?

| **Cache**                                 | **Artifacts**                                                |
|---------------------------------------|----------------------------------------------------------|
| Internal Storage                      | External Storage                                         |
| Save precompiled assets, node modules | Save test coverage, logs                                 |
| Shared between runs                   | Not shared between runs                                  |
| Deleted when unused for 7 days        | Deleted after 90 days(or specified in retention options) |

* * *

## Properly cache your Webpacker files
Skip if you aren't using **feature tests and Webpacker** in your project.

Feature tests always take more time after changes to any of the Javascript files. It's because feature tests require asset precompilation, on your local machine it usually takes only a few seconds so might not notice it. But the runners that GitHub Actions are using are way slower. To the degree that you shouldn't be surprised if the asset precompilation takes minutes.

But you aren't changing the Javascript files for every CI run, right? With proper cache utilization, you should be able to avoid asset precompilation in most cases.

If you are using Webpacker(or now [Shakapacker](#TODO)) then I got you covered, add this action before the `rails assets:precompile` action:
```yaml
- name: Get Webpacker cache key
  id: get-webpacker-cache-key
  run: |
    echo "::set-output name=key::$(bin/rails r 'print Webpacker::Instance.new.compiler.send(:watched_files_digest)')"
  shell: bash

- name: Cache precompiled assets
  uses: actions/cache@v3
  with:
    path: |
      public/packs
      tmp/cache/webpacker
    key: ${{ runner.os }}-assets-${{ steps.get-webpacker-cache-key.outputs.key }}
    restore-keys: |
      ${{ runner.os }}-assets-
```

It's getting `digest` from your Webpacker configuration which is basically a unique identifier of the current state of your Javascript files. If any of the files would change then the `digest` would be also different.

It's also worth mentioning that **you should always cache the `public_output_path` defined in `config/webpacker.yml` file.**

I've seen multiple projects where it wasn't done properly which cost hours of additional asset precompilation that could be avoided.

One last note: Some people tend to not put explicitly `rails assets:precompile` in the configuration as `rails test` or `rspec` command will compile the assets anyway. However, I find it useful to have a separate pipeline step for asset precompilation. This way the precompilation won't pollute the time results of your test action.

## Feature Tests
### Use proper Chromium command line switches:
```ruby
# Usually it'd be configured this way somewhere in your test folder
chrome_options = Selenium::WebDriver::Chrome::Options.new
chrome_options.add_argument("--headless")
chrome_options.add_argument("--disable-gpu")
Capybara::Selenium::Driver.new(app, browser: :chrome, capabilities: [options])
```
`--disable-gpu`: Github Actions runner doesn't have GPU so we should tell Chrome not to try using it for page rendering, that might be the biggest difference maker from all 3
`--no-sandbox`: is making sure Chrome sandbox mode isn't used as it's not needed in the CI environment
`--blink-settings=imagesEnabled=false`: as long as you aren't running visual regressions tests it should be safe to disable image render.

There are [literally thousands](https://peter.sh/experiments/chromium-command-line-switches/) of those switches, if you want to optimize your tests by changing the browser behaviour there are high chances there is an switch for it.

### Check your Capybara.default_max_wait_time:
There is no correct wait time, but in most cases, it should be no longer than a 5-10 seconds, for special cases it's possible to use such blocks in your tests:
```ruby
using_wait_time x do # where x is the number of seconds
	# Test logic goes here
end
```

### Use request proxy - Puffing Billy
Feature tests are usually loading a lot of 3rd party stuff like tracking scripts, google fonts or any other unrelevant for our test environment stuff. **If you think:**

**"I do have WebMock blocking internet connections so it's not for me"**

**you are wrong. WebMock can't block requests done by the browser in feature tests, Puffing Billy can.** Moreover, besides blocking unwanted resources it could also cache any front-end-side request that is actually relevant.

It takes a good amount of time to set up it in a way that everyone benefits from it, especially in large projects. But being used properly is a powerful tool.

**Setup**

Add proxy server to your driver initialization:
```ruby
chrome_options.add_argument("--proxy-server=#{Billy.proxy.host}:#{Billy.proxy.port}")
```
Configure Billy:
```ruby
# spec/rails_helper.rb
Billy.configure do |c|
  c.cache = true
  c.persist_cache = true
  c.cache_path = "spec/req_cache/"

  c.non_whitelisted_requests_disabled = true
  c.ignore_params = [ # Do not match request parameters
    'https://maps.googleapis.com/maps/vt',
    'https://widget.trustpilot.com/stats/TrustboxImpression',
    ...
  ]
  c.cache_request_body_methods = [] # Do not match POST request body
end
```
You could add your cache_path to `.gitignore` file and add pipeline caching to this folder. Or - it's a messier approach in my point of view: store all of the cached requests in your repository. The latter allows you to benefit from Puffing Billy also in your local environment.

## Optimize Test Suite
No matter how performant will be our configuration, if our tests are extremely slow then we won't see any improvements.

As a starter point I'd use `rspec --profile` command that will show us the 10 slowest examples and also the test groups:
```ruby
# some example output showcasing that we might focus on exceptionally slow examples
```

The `--profile` flag is only scratching the surface there.

#### Test-prof gem
To obtain real metrics I recommend that everyone use the [test-prof gem](https://test-prof.evilmartians.io/#/) as it seems to be extremely powerful tool for benchmarking and optimizing your test suite. There is a great article that you'd find on the gem homepage: https://evilmartians.com/chronicles/testprof-a-good-doctor-for-slow-ruby-tests that describes most of the usecases. It's a great starting point.

> Spending a few minutes on learning how to benchmark your test suite with test-prof might be one of the highest-leverage activities I did this year

If you are concerned about your feedback loop then it's a must to go through the stuff that Evlimartians prepared for us!

A small cheatsheet of commands described in the articles:
```sh
TAG_PROF=type rspec # find out which type of tests is the slowest for you

FDOC=1 rspec # highlight possible usages of factory `build` instead of `create`

FPROF=1 bundle exec rspec # number of times each factory is called

EVENT_PROF="factory.create" bundle exec rspec # time spent on factory creation

TEST_STACK_PROF=1 rspec && stackprof tmp/test_prof/stack-prof-report-wall-raw-total.dump # where the most time were spent in terms of called classes

EVENT_PROF=sidekiq.inline rspec # check whether sidekiq worker executions aren't slowing you tremendously
```

#### Flaky tests
A test would be considered "flaky" once it's result isn't deterministic. Maybe it's waiting for external service and timeout from time to time? Or it fails when the order of running the tests is different?

I'd consider that flaky test is basically a failing test. The immediate action could be to mark every unstable example as skipped with proper comment.

Sometimes finding out why our tests are flaky isn't a friendly task and could take a good amount of time. Having a test that could give us false positives might be even worse.

Some projects will use gems like `rspec-retry` to make sure the pipeline success even with flaky tests. But retrying the examples takes time so it's definitely a good idea to take action.

## Parallelization

TODO: Add the screenshots

Parallelization is the last step as without adding all of the configurations tweaks and no proper cache usage - it will only spread the slowness of your pipeline to multiple chunks.

**Minitest support paralleization out of the box: ADD LINK**

#### Things to know
- Our goal should be to always have all of our pipelines running for the same amount of time. Situation when one chunk of tests is executed `2 minutes` and another one takes `5 minutes` is something we want to avoid. In the given example ideally, both actions take around `3 minutes`.
- The configuration for each pipeline action shouldn't take longer than the actual tests. Otherwise it might be a signal that we're over paralyzed ;)

#### Meet [Parallel_tests](https://github.com/grosser/parallel_tests) gem
-   Automatically splits tests into groups
-   Generates file runtime stats for all tests*
-   *we have to somehow provide the runtimes file to CI 
-   Easy to scale

**Setup**

-   Add _TEST_ENV_NUMBER_ to your database configuration
-   Start your tests with command like:
```sh
parallel_test spec/ -n $NO_GROUPS --only-group $GROUP --group-by runtime
```
-   Github Actions setup:
```yaml
strategy:
		fail-fast: false
		matrix:
		ci_node_total: [4]
		ci_node_index: [0, 1, 2, 3]

	- name: "Run tests"
		env:
			NO_GROUPS: ${ { matrix.ci_node_total } }
			GROUP: ${ { matrix.ci_node_index } }
```


https://github.com/Scandotcom/nmri/pull/306
https://github.com/Scandotcom/nmri/pull/318
