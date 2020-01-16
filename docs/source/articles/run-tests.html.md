---
title: Run Tests
created_at: 2020/01/13
excerpt: TODO
---

# Run Tests

[Testing Overview, Tests & Decorators](/articles/testing-overview.html#tests-decorators)

* [Run All Tests](#run-all-tests)
* [Run Workarea Tests](#run-workarea-tests)
* [Run Application Tests](#run-application-tests)

* [List Test Runners](#list-test-runners)
* [Display Rails Test Runner Help](#display-rails-test-runner-help)
* [Pass Arguments to Workarea Test Runners](#pass-arguments-to-workarea-test-runners)


## Run All Tests

Uses a workarea test runner.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)

Runs the workarea platform tests.
With any decorators you've written applied.
Any any new tests original to your application.

```
bin/rails workarea:test
```

Uses the rails test runner.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)
Passes paths of `/test/**/*_test.rb` within the installed
Workarea Core and each installed plugin
And rails root

[Pass Arguments to Workarea Test Runners](#pass-arguments-to-workarea-test-runners)


## Run Workarea Tests

Uses a workarea test runner.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)

Run only Workarea platform tests.
Any decorators for these tests are applied.

By engine:

```
bin/rails workarea:test:<engine>
```

e.g.:

```
bin/rails workarea:test:core
bin/rails workarea:test:storefront
bin/rails workarea:test:gift_cards
```

the special case `plugins` runs all engines
except core, admin, storefront

```
bin/rails workarea::test:plugins
```

[Pass Arguments to Workarea Test Runners](#pass-arguments-to-workarea-test-runners)


### Run Specific Workarea Tests

Uses the rails test runner.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)

Use the Rails test runner to run individual/specific Workarea tests.
Pass the pathnames of the workarea test files.
Use `bundle show` to easily find the installation location of each workarea engine.

Run `UserTest` test case from Core:

```
bin/rails test $(bundle show workarea-core)/test/models/workarea/user_test.rb
```

Use any args supported by the rails test runner, e.g. `-n` to run specific tests within a test case:

```
bin/rails test $(bundle show workarea-core)/test/models/workarea/user_test.rb -n test_new_example
```

[Display Rails Test Runner Help](#display-rails-test-runner-help)


### Run Decorated Workarea Tests

Uses a workarea test runner.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)

```
bin/rails workarea:test:decorated
```

finds the originals that correspond to your application decorators
within core and installed plugins and runs them.

Run single decorated test just like you would run any other single test.
[Run Specific Workarea Tests](#run-specific-workarea-tests)

Common mistake is to try to pass a decorator as an argument to the test runner.
Have to find the path to the original test.
pass pathnames ending in `.rb`, not `.decorator`

[Pass Arguments to Workarea Test Runners](#pass-arguments-to-workarea-test-runners)


## Run Application Tests

Uses the rails test runner.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)

```
bin/rails test
```

The default Rails test runner doesn't run system tests, which trips people up.

```
bin/rails test:system
```

[Display Rails Test Runner Help](#display-rails-test-runner-help)


### Run Specific Application Tests

Uses the rails test runner.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)

```
bin/rails test <paths>
```

e.g.:

```
bin/rails test test/models/article_test.rb
```

[Display Rails Test Runner Help](#display-rails-test-runner-help)


### Run `.decorator` Test Files

Not really a thing. See other section.
[Run Decorated Workarea Tests](#run-decorated-workarea-tests)


## List Test Runners

List all available test runners, including rails and workarea.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)

```
bin/rails -T test
```


## Display Rails Test Runner Help

```
bin/rails test --help
```

## Pass Arguments to Workarea Test Runners

Workarea test runners accept same options as Rails test runner, but using ENV var.
[Display Rails Test Runner Help](#display-rails-test-runner-help)

e.g. `-b` to get a backtrace

v is verbose
s is seed

```
$ TESTOPTS='-v -s 51477' bin/rails workarea:test:decorated
```
