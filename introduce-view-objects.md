# Introduce View Objects

<http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/>

## 適合場景

如果一段邏輯只是為了展示 View，而不屬於 Model 裡的業務邏輯。可以考慮放在 Helper，或者乾脆抽出去變成 View object。


``` ruby

class DonutChart
  def initialize(snapshot)
    @snapshot = snapshot
  end

  def cache_key
    @snapshot.id.to_s
  end

  def data
    # pull data from @snapshot and turn it into a JSON structure
  end
end

```

## Futher Reading

* [Presenter](http://joncanady.com/blog/2012/01/11/presenters-cleaning-up-rails-views/)

## My Example

`before`

`app/helper/registrtions_helper.rb`

``` ruby
module RegistrationsHelper
  def render_registrant_status(registrant)
    I18n.t("registrants_status.#{registrant.payment_state}")
  end
end

```

`app/models/registrant.rb`

``` ruby

class Registrant < ActiveRecord::Base
  def payment_state
    if paid_at && cancelled_at
      "paid_but_cancelled"
    end

    if cancelled_at
      "cancelled"
    else
      if paid_at
        "paid"

      else
        "not_paid"
      end
    end
  end
end

```


`after`

`app/helper/registrtions_helper.rb`

``` ruby
module RegistrationsHelper
  def render_registrant_status(registrant)
    RegistrantPaymentState.new(registrant).chinese_payment_state
  end
end

```

`app/decorators/registrant_payment_state.rb`

``` ruby
class RegistrantPaymentState
  def initialize(registrant)
    @registrant = registrant
  end

  def payment_state
    if @registrant.paid_at && @registrant.cancelled_at
      "paid_but_cancelled"
    end

    if @registrant.cancelled_at
      "cancelled"
    else
      if registrant.paid_at
        "paid"

      else
        "not_paid"
      end
    end
  end

  def chinese_payment_state
    I18n.t("registrants_status.#{payment_state}")
  end
end

```

