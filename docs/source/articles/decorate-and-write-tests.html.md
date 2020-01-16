---
title: Decorate & Write Tests
created_at: 2020/01/13
excerpt: TODO
---

# Decorate & Write Tests


## Decorate a Workarea Test

When to decorate a test, see [Testing Overview, Tests & Decorators](/articles/testing-overview.html#tests-amp-decorators)

[Decorate](/articles/decoration.html) a test case just like any other ruby class, except the pathname of the decorator must match the pathname of the original file
except for the file extension

[Testing Overview, Test Case Types & Mixins](/articles/testing-overview.html#test-case-types-amp-mixins)

```
https://github.com/workarea-commerce/workarea/blob/master/core/test/models/workarea/payment/credit_card_integration_test.rb
https://github.com/workarea-commerce/workarea-braintree/blob/master/test/models/workarea/payment/credit_card_integration_test.decorator
```

```
workarea-core/test/models/workarea/payment/credit_card_integration_test.rb
workarea-braintree/test/models/workarea/payment/credit_card_integration_test.decorator
```

This is necessary for the decorator to be applied to the class.


## Skip a Workarea Test

Special case of decorating a test that is useful and overlooked.

```
# board-game-supercenter/test/system/workarea/storefront/users/hearts_system_test.decorator

require 'test_helper'

module Workarea
  decorate Storefront::Users::HeartsSystemTest do
    # skip all tests in this test case
    decorated { setup :skip }
  end
end

# board-game-supercenter/test/system/workarea/admin/inventory_skus_system_test.decorator

require 'test_helper'

module Workarea
  decorate Admin::InventorySkusSystemTest do
    def test_editing_a_non_existent_sku
      skip('removed this feature')
    end
  end
end

# board-game-supercenter/test/system/workarea/admin/publish_authorization_system_test.decorator

require 'test_helper'

module Workarea
  decorate Admin::PublishAuthorizationSystemTest do
    def test_user_cannot_select_publish_now_in_workflows
      skip('defer until custom permission is implemented')
    end

    def test_user_cannot_submit_form_without_selecting_a_release
      skip('defer until custom permission is implemented')
    end
  end
end

# board-game-supercenter/test/documentation/workarea/api/storefront/checkouts_documentation_test.decorator

require 'test_helper'

module Workarea
  decorate Api::Storefront::CheckoutsDocumentationTest do
    def test_and_document_complete
      skip('remove until gift card upgrade')
    end
    def test_and_document_update
      skip('remove until gift card upgrade')
    end
    def test_and_document_reset
      skip('remove until gift card upgrade')
    end
    def test_and_document_show
      skip('remove until gift card upgrade')
    end
  end
end

# board-game-supercenter/test/models/workarea/payment/refund/credit_card_test.decorator

require 'test_helper'

module Workarea
  decorate Payment::Refund::CreditCardTest do
    def test_complete_refunds_on_the_credit_card_gateway
      skip('skip until gateway bug is resolved')
    end
  end
end
```


## Write an Application Test

Prefer new tests over decorators.
When to decorate a test, see [Testing Overview, Tests & Decorators](/articles/testing-overview.html#tests-amp-decorators)

To create a test, create a new ruby file to represent the test case.
Choose a pathname based on workarea conventions.
Refer to workarea engines to see how tests are grouped.
But always within `/test`.

```
https://github.com/workarea-commerce/workarea/tree/master/core/test
https://github.com/workarea-commerce/workarea/tree/master/storefront/test
```

require the test helper
define a class
inherit from the appropriate test case type
mix in additional test support

[Test Case Types & Mixins](/articles/testing-overview.html#test-case-types-amp-mixins)

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


## Change Configuration within a Test

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


## Change a Locale within a Test

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

## Change Time within a Test

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

## Conditionally Define a Test

As a plugin author, you can't control the environment in which your plugins' tests are run. It is therefore useful to define some tests only when certain conditions are met, such as another particular plugin being installed or optional code being present in the environment. The following examples demonstrate the concept of conditionally defining tests.

The following examples from Browse Option, Clothing, and Gift Cards demonstrate conditionally defining tests when another plugin is installed.

```
# workarea-browse_option-1.2.1/test/integration/workarea/api/storefront/browse_option_product_integration_test.rb

require 'test_helper'

module Workarea
  module Api
    module Storefront
      class BrowseOptionProductIntegrationTest < Workarea::IntegrationTest
        if Plugin.installed?('Workarea::Api::Storefront')
          setup :product, :category, :index_product

          def product
            # ...
          end

          def category
            # ...
          end

          def index_product
            # ...
          end

          def test_category_show
            # ...
          end

          def test_product_show
            # ...
          end
        end
      end
    end
  end
end
```

```
# workarea-clothing-2.1.1/test/integration/workarea/api/storefront/product_swatches_integration_test.rb

require 'test_helper'

module Workarea
  module Api
    module Storefront
      class ProductSwatchesIntegrationTest < Workarea::IntegrationTest
        if Plugin.installed?('Workarea::Api::Storefront')
          setup :set_product

          def set_product
            # ...
          end

          def test_shows_products
            # ...
          end
        end
      end
    end
  end
end
```

```
# workarea-gift_cards-3.2.0/test/integration/workarea/api/storefront/balance_integration_test.rb

require 'test_helper'

module Workarea
  if Plugin.installed?(:api)
    module Api
      module Storefront
        class BalanceIntegrationTest < Workarea::IntegrationTest
          def test_balance_lookup
            # ...
          end
        end
      end
    end
  end
end
```

The following examples from Address Verification wrap test definitions in two other types of conditionals. The first, Workarea::TestCase.running_in_gem? is true only when the test case is run from the engine's embedded "dummy" app. This allows defining tests that are useful to the plugin maintainers but may be problematic to include in the test suites of applications that install the plugin.

The other, Workarea.const_defined? tests the environment for the presence of a particular constant before defining tests. Address Verification supports multiple address verification gateways (and therefore provides tests for multiple gateways), but only one gateway will be installed in a production app. The conditional ensures tests for only the installed gateway are run.

```
# workarea-address_verification-2.0.2/test/lib/workarea/address_verification/ups_gateway_test.rb

require 'test_helper'

if Workarea::TestCase.running_in_gem? ||
   Workarea.const_defined?('AddressVerification::UpsGateway')

  require 'workarea/address_verification/ups_gateway'

  module Workarea
    module AddressVerification
      class UpsGatewayTest < TestCase
        def test_verify
          # ...
        end
      end
    end
  end
end
```

```
# workarea-address_verification-2.0.2/test/lib/workarea/address_verification/melissa_data_gateway_test.rb

require 'test_helper'

if Workarea::TestCase.running_in_gem? ||
   Workarea.const_defined?('AddressVerification::MelissaDataGateway')

  require 'workarea/address_verification/melissa_data_gateway'

  module Workarea
    module AddressVerification
      class MelissaDataGatewayTest < TestCase
        def test_verify
          # ...
        end
      end
    end
  end
end
```
