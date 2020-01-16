---
title: Testing Overview
created_at: 2020/01/13
excerpt: TODO
---

# Testing Overview


## Tests & Decorators

When to write a new test (an application test) vs decorating an existing test (a workarea test).
Prefer writing new tests.
Decorate only when you have to.
Examples of when you have to are when platform extensions you wrote break an existing
platform test, so you have to extend the test to make it pass (or to simply skip the test)
you need to change the vcr cassette that's used

```
https://github.com/workarea-commerce/workarea-braintree/blob/master/test/models/workarea/payment/store_credit_card_test.decorator
```

Less cognitive overhead to write and run new tests and see pathnames for new tests.
Decorators can be a little confusing to work with (e.g have to run the original file).

Workarea (platform) tests vs application tests.

Application tests vs decorators.

Link to doc that covers decorating and writing tests.
[Decorate & Write Tests](/articles/decorate-and-write-tests.html)


## Test Runners

[Rails Test Runner](https://guides.rubyonrails.org/testing.html#the-rails-test-runner)

Rails test runner vs Workarea test runners.

Link to doc that covers running tests.
[Run Tests](/articles/run-tests.html)

With workarea test runner, when tests fail, the output includes a command line to re-run each failed test (with the Rails test runner).


## Test Case Types & Mixins

All Workarea test cases inherit from one of the test case types below. The test case types generally differ in the size and scope of the Ruby API available within the test case, and the amount and type of setup and teardown that's done before/after each test in the test case.

For example, Workarea::SystemTest provides a staggering 584 instance methods and starts up a headless browser to interact with the running application before each test. This provides greater test fidelity and flexibility than other test types, but slower performance. Meanwhile, Workarea::GeneratorTest provides a smaller Ruby API specialized for testing generators (scripts) and does not perform any setup.

I summarize each of the test case types below. All of the test types below extend Workarea::TestCase::Decoration, which allows applications and plugins to decorate the tests within that test case (and test cases that inherit from it).
Generic Tests

Generic test cases inherit from Workarea::TestCase.

Among the ancestors of Workarea::TestCase are ActiveSupport::TestCase, Minitest::Test, Minitest::Assertions, ActiveSupport::Testing::Assertions, and ActiveSupport::Testing::TimeHelpers. The class's instance methods include running_in_gem?, which is also available as the class method Workarea::TestCase.running_in_gem?.
Integration Tests

Test cases inheriting from Workarea::IntegrationTest are testing how various parts of the platform and/or application are interacting.

Workarea::IntegrationTest inherits directly from ActionDispatch::IntegrationTest. Beginning with Workarea 3.1.0, Workarea::IntegrationTest includes Workarea::IntegrationTest::Configuration, a module used to share behavior with system tests. The class's instance methods include set_current_user.
System Tests

Test cases inheriting from Workarea::SystemTest use a headless browser to interact with the application's UI the same way a user does. Workarea 3.2.0 and below used PhantomJS as a headless browser for running system tests, but Workarea 3.3.0 uses the Chrome webdriver for Selenium ("headless" Chrome) by default, which does not include a browser cache. Workarea 3.3.0 still depends on Poltergeist (the Capybara driver for PhantomJS) for upgrading applications that may still depend on facilities provided by PhantomJS, but all newly-written tests should execute under the Headless Chrome driver.

The ancestors of Workarea::SystemTest include ActionDispatch::SystemTestCase (since Workarea 3.1.0), Workarea::IntegrationTest::Configuration (since Workarea 3.1.0), and Capybara::DSL. Prior to Workarea 3.1.0, Workarea::SystemTest inherited directly from Workarea::IntegrationTest. Workarea 3.1.0 depends on Rails 5.1, which adds ActionDispatch::SystemTestCase, so Workarea::SystemTest inherits from that class instead. The instance methods for SystemTest include t, clear_driver_cache, and wait_for_xhr.

Workarea also extends the Capybara DSL with the #has_ordered_text? method, available on element objects as well as generically in test case. All methods that manipulate the DOM will also call #wait_for_xhr before returning back to the callee, ensuring Ajax requests have finished before proceeding.Link to reference docs for these classes and modules.

View Tests

Test cases inheriting from Workarea::ViewTest provide the necessary API to test view helpers.

Workarea::ViewTest inherits directly from ActionView::TestCase. The class's instance methods include all of Rails' view helpers.
Generator Tests

Test cases inheriting from Workarea::GeneratorTest provide an API suitable for testing the Rails generators included with Workarea.

Workarea::GeneratorTest inherits directly from Rails::Generators::TestCase.

Link to doc that covers extension of these.
[Add or Change Test Factories & Support](/articles/add-or-change-test-factories-and-support.html)

Additional Setup/Teardown

The following modules provide additional test setup and/or teardown.

    Workarea::TestCase::Workers
        Setup and teardown for workers
        Included in Workarea::TestCase, Workarea::IntegrationTest, and Workarea::SystemTest by default
    Workarea::TestCase::SearchIndexing
        Setup for Elasticsearch indexes
        Included in Workarea::IntegrationTest and Workarea::SystemTest by default
    Workarea::Storefront::CatalogCustomizationTestClass
        Setup and teardown for catalog customization tests

Tests for Shared Behavior

The following modules provide tests for shared behavior. When included in a test case, each module provides the tests indicated below.

    Workarea::DiscountConditionTests::OrderTotal
        test_order_total?
        test_order_total_qualifies?

    Workarea::DiscountConditionTests::PromoCodes
        test_promo_codes_qualify?

    Workarea::DiscountConditionTests::ItemQuantity
        test_item_quantity?
        test_items_qualify?

    Workarea::Storefront::PaginationViewModelTest
        test_total_pages
        test_first_page
        test_last_page
        test_next_page
        test_prev_page

    Workarea::Storefront::ProductBrowsingViewModelTest
        test_has_filters
        test_facets

Test Helpers

The following modules provide additional instance methods that are useful in certain test situations and are usually referred to as test helpers or macros. I've listed the helper methods for each available module below.

    Workarea::Admin::IntegrationTest
        admin_user (Also sets current_user to this user as additional test setup)
    Workarea::Storefront::IntegrationTest
        complete_checkout
        product
    Workarea::Storefront::SystemTest
        add_product_to_cart
        add_user_data
        create_supporting_data
        fill_in_billing_address
        fill_in_credit_card
        fill_in_email
        fill_in_new_card_cvv
        fill_in_shipping_address
        select_shipping_service
        setup_checkout_specs
        start_guest_checkout
        start_user_checkout
    Workarea::BreakpointHelpers
        resize_window_to


## Test Help

Every test case file begins by requiring your application test helper, test/test_helper.rb. This file sets up the environment for testing, as follows:

1. Boots your application in the test Rails environment
2. Loads Rails' test help, railties/lib/rails/test_help.rb, which bootstraps Rails testing
3. Loads Workarea's test help, workarea-testing/lib/workarea/test_help.rb, which bootstraps Workarea testing

The Workarea test help file loads a large Ruby API into memory, including the Workarea test cases from which your own test cases inherit. However, it also includes many other modules that provide additional setup/teardown and instance methods to use within your tests. Many of the platform's test cases mix in these modules as needed, and they are available to mix into your own test cases as well.


## Headless Chrome

Link to doc that covers its configuration.
[Configure Headless Chrome](/articles/configure-headless-chrome.html)


## Factories

Link to docs that cover configuration and adding new ones.
[Add or Change Test Factories & Support](/articles/add-or-change-test-factories-and-support.html)

Factories are specialized test helpers that provide shortcuts for creating model instances. A factory method, such as create_product, can generally be called without arguments and creates a model instance using default data appropriate for testing.

Core factories are organized into files under workarea-testing/lib/workarea/testing/factories/. However, test cases that use factories mix in only Workarea::Factories. Including this module automatically includes all factory methods from all factory modules in Core and in all installed plugins.

The following test case types include factories by default.

    Workarea::TestCase
    Workarea::IntegrationTest
    Workarea::SystemTest
    Workarea::ViewTest

Including factories in a test case adds the following instance methods.

    complete_checkout
    ...
