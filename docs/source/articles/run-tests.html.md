---
title: Run Tests
created_at: 2020/01/13
excerpt: TODO
---

# Run Tests

The platform ships with its own tests (Workarea tests), which you can run from your application.
When you run the platform tests, any decorators you've written for those tests are applied.
And you can also write your own tests (application tests) that further extend the platform.
See also [Testing Overview, Tests & Decorators](/articles/testing-overview.html#tests-decorators)

You run the tests using one of many test runners.
See also [Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)

It can be confusing which to use for a given situation, so this document provides instructions to:

* [Run All Tests](#run-all-tests)
* [Run Workarea Tests](#run-workarea-tests)
* [Run Application Tests](#run-application-tests)

Once you are familiar with these procedures, you may only need a reminder of specific details.
In those situations you may want to refer to:

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

Under the hood, this uses the rails test runner.
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

To run _all_ the tests, you've decorated from your application,
use the following command.
Uses a workarea test runner.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)
finds the originals that correspond to your application decorators
within core and installed plugins and runs them.

```
bin/rails workarea:test:decorated
```
[Pass Arguments to Workarea Test Runners](#pass-arguments-to-workarea-test-runners)

To run _specific_ tests you've decorated...
Run specific decorated tests just like you would run any other single test.
[Run Specific Workarea Tests](#run-specific-workarea-tests)
Common mistake is to try to pass a decorator as an argument to the test runner.
Have to find the path to the original test.
pass pathnames ending in `.rb`, not `.decorator`


## Run Application Tests

Nothing Workarea-specific about this.
Use the standard Rails test runner like you would in any other Rails app.
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

Again, nothing Workarea specific.
Use the standard Rails test runner, and pass it pathnames as arguments.
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

It's a common mistake to try to run `.decorator` files using the Rails test runner.
Remember that [decorators](https://developer.workarea.com/articles/decoration.html#decorators) are files that extend an existing class.
So they aren't complete test classes.
To run a decorated test, you have to pass the pathname of the original test case.
Refer to [Run Decorated Workarea Tests](#run-decorated-workarea-tests)


## List Test Runners

Once you are familiar with the commands from the sections above, you may simply need a refesher of which
test runners exist.
List all available test runners, including rails and workarea.
[Testing Overview, Test Runners](/articles/testing-overview.html#test-runners)

```
bin/rails -T test
```

Output will differ from app to app, depending on which plugins are installed and possibly other things.


## Display Rails Test Runner Help

To see all the arguments accepted by the Rails test runner, view its inline help.

```
bin/rails test --help
```

e.g. `-b` to get a backtrace

v is verbose
s is seed


## Pass Arguments to Workarea Test Runners

Workarea test runners accept same options as Rails test runner
[Display Rails Test Runner Help](#display-rails-test-runner-help)
but using ENV var.

Begin your command line with a variable definition containing the options.

```
TESTOPTS='<arguments>' bin/rails workarea:test:<runner>
```

e.g., run all core tests with verbose output using a specific seed:

```
TESTOPTS='-v -s 51477' bin/rails workarea:test:core
```

Or, run all decorated tests with backtraces enabled for failed tests:

```
TESTOPTS='-b' bin/rails workarea:test:decorated
```
