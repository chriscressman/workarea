---
title: Extend Test Case Mixins
created_at: 2020/01/13
excerpt: TODO
---

# Extend Test Case Mixins

[Testing Concepts, Test Case Mixins](/articles/testing-concepts.html#test-case-mixins)

* [Change an Existing Test Case Mixin](#change-an-existing-test-case-mixin)
* [Add a New Test Case Mixin](#add-a-new-test-case-mixin)


## Change an Existing Test Case Mixin

e.g. want to change the email filled in by:

[`Storefront::SystemTest#fill_in_email`](https://github.com/workarea-commerce/workarea/blob/v3.5.3/testing/lib/workarea/storefront/system_test.rb#L90-L92)

Change email from:

bcrouse-new-account@workarea.com

To:

robert-clams@workarea.com

Create a new file in /test/support/ and re-open the module and the method:

```
# <application_root>/test/support/storefront_system_test_extensions.rb

module Workarea
  module Storefront
    module SystemTest

      def fill_in_email
        fill_in 'email', match: :first, with: 'bcrouse-new-account@workarea.com'
      end
    end
  end
end
```

Require the new method from your test helper:

```
# <application_root>/test/test_helper.rb

ENV['RAILS_ENV'] ||= 'test'

# ...

require 'support/storefront_system_test_extensions'
```

## Add a New Test Case Mixin

Follow the steps for changing a test case mixin, but define a new module and methods instead of re-opening an existing module and methods.
Now you can mix the module into new test cases and use the new methods.
