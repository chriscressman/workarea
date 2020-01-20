---
title: Decorate & Write Tests
created_at: 2020/01/13
excerpt: TODO
---

# Decorate & Write Tests

From your application, you can decorate the tests that ship with Workarea,
and you can write your own application tests.
See [Testing Concepts, Tests & Decorators](/articles/testing-concepts.html#tests-amp-decorators)
for more info, including an explanation of when to decorate vs when to write your own.

This doc explains how to
[Decorate a Workarea Test Case](#decorate-a-workarea-test-case)
including the special case
[Skip a Workarea Test](#skip-a-workarea-test)

And also how to
[Write an Application Test Case](#write-an-application-test-case)

When doing so, the following recipes may also be useful:

* [Change Configuration within a Test](#change-configuration-within-a-test)
* [Change Locale within a Test](#change-locale-within-a-test)
* [Change Time within a Test](#change-time-within-a-test)


## Decorate a Workarea Test Case

[Decorate](/articles/decoration.html) a test case just like any other ruby class, except the pathname of the decorator must match the pathname of the original file
except for the file extension

[Testing Concepts, Test Case Types & Mixins](/articles/testing-concepts.html#test-case-types-amp-mixins)

e.g., to decorate the class definition:

```
workarea-core/test/models/workarea/payment/credit_card_integration_test.rb
```

create the decorator:

```
workarea-braintree/test/models/workarea/payment/credit_card_integration_test.decorator
```

This is necessary for the decorator to be applied to the class when tests are run.


### Skip a Workarea Test

One reason to decorate a test case is to skip one or more tests.
Makes sense to skip a test when your extensions make it irrelevant or not applicable.
e.g. you remove the ability to manage addresses in workarea as part of an
integration with a user management system
or temporarily removed it until while working on a related feature
that will affect it
or skip because there is a platform bug and you're waiting for patch

Skip all tests within a test case:

```
# <your_application>/test/system/workarea/storefront/addresses_system_test.decorator

require 'test_helper'

module Workarea
  decorate Storefront::AddressesSystemTest do
    decorated { setup :skip }
  end
end
```

Skip specific tests within a test case:

```
# <your_application>/test/system/workarea/storefront/addresses_system_test.decorator

require 'test_helper'

module Workarea
  decorate Storefront::AddressesSystemTest do
    def test_managing_addresses
      skip('defer until customer user management is implemented')
    end
  end
end
```

```
# <your_application>/test/models/workarea/payment/refund/credit_card_test.decorator

require 'test_helper'

module Workarea
  decorate Payment::Refund::CreditCardTest do
    def test_complete_refunds_on_the_credit_card_gateway
      skip('skip until gateway bug is resolved')
    end
  end
end
```


## Write an Application Test Case

Prefer new tests over decorators.
When to decorate a test, see [Testing Concepts, Tests & Decorators](/articles/testing-concepts.html#tests-amp-decorators)

To create a test, create a new ruby file to represent the test case.
Choose a pathname based on workarea conventions.
Refer to workarea engines to see how tests are grouped.
But always within `/test`.

[`/test` directory for Workarea Core](https://github.com/workarea-commerce/workarea/tree/master/core/test)

[`/test` directory for Workarea Storefront](https://github.com/workarea-commerce/workarea/tree/master/storefront/test)

require the test helper
define a class
inherit from the appropriate test case type
mix in additional test support

[Test Case Types & Mixins](/articles/testing-concepts.html#test-case-types-amp-mixins)

Example. Ideally replace this with a better one:

```
# test/workers/workarea/import_inventory_test.rb

require 'test_helper'

module Workarea
  class ImportInventoryTest < TestCase

    def test_perform
      # ...
    end
  end
end
```


### Change Configuration within a Test

Since Workarea 3.5.0, the global configuration is reset before each test. You can therefore change the configuration within a test and have it affect only that test. Here is an example from Workarea Core:

```
module Workarea
  class UserTest < TestCase
    def test_admins_have_more_advanced_password_requirements
      config.password_strength = :weak

      user = User.new(admin: false, password: 'password').tap(&:valid?)
      assert(user.errors[:password].blank?)

      user = User.new(admin: true, password: 'password').tap(&:valid?)
      assert(user.errors[:password].present?)

      user = User.new(admin: true, password: 'xykrDQXT]9Ai7XEXfe').tap(&:valid?)
      assert(user.errors[:password].blank?)
    end
  end
end
```

Prior to Workarea 3.5.0, you must wrap configuration changes in Workarea.with_config to ensure they reset after the test. Here is the same example from above using Workarea.with_config:

```
module Workarea
  class UserTest < TestCase
    def test_admins_have_more_advanced_password_requirements
      Workarea.with_config do |config|
        config.password_strength = :weak

        user = User.new(admin: false, password: 'password').tap(&:valid?)
        assert(user.errors[:password].blank?)

        user = User.new(admin: true, password: 'password').tap(&:valid?)
        assert(user.errors[:password].present?)

        user = User.new(admin: true, password: 'xykrDQXT]9Ai7XEXfe').tap(&:valid?)
        assert(user.errors[:password].blank?)
      end
    end
  end
end
```


### Change Locale within a Test

It's also possible to change the locale for the duration of a test, using the I18n.with_locale method. This is the method used to change locale in Workarea::I18nServerMiddleware, but it's also useful within tests like so:

```
module Workarea
  decorate UserTest do
    def test_title
      user = create_user

      I18n.with_locale :en do
        assert_equal 'Mister', user.title
      end

      I18n.with_locale :es do
        assert_equal 'Señor', user.title
      end
    end
  end
end
```

### Change Time within a Test

In previous versions of Workarea, the Timecop gem was used to simulate running code at different points in time. Since Workarea 3.0.0, ActiveSupport's Time Helpers methods (like travel_to) are used for changing the current time and date. Note that changes to the current time will not carry over to other tests, Time.current is reset to the actual current time of the machine after executing each test. Here's an example using travel_to within a unit test to see how data is presented over time:

https://api.rubyonrails.org/v5.2/classes/ActiveSupport/Testing/TimeHelpers.html

```
module Workarea
  module Analytics
    class DailyDataTest < TestCase
      def test_days_ago_index
        travel_to '2017/2/19'.in_time_zone(Workarea.config.analytics_timezone)
        assert_equal(6, DailyData.days_ago_index(1))
        assert_equal(5, DailyData.days_ago_index(2))

        travel_to '2017/2/20'.in_time_zone(Workarea.config.analytics_timezone)
        assert_equal(0, DailyData.days_ago_index(1))
        assert_equal(6, DailyData.days_ago_index(2))
      end
    end
  end
end
```
