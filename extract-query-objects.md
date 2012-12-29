# Extract Query Objects

<http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/>

## 適合場景

如果有一些複雜的 scope 或者是 class method 放在 model 裡。可以試著搬出 ActiveRecord ，變成 Query Objects。


例如 `AbandonedTrialQuery`


``` ruby

class AbandonedTrialQuery
  def initialize(relation = Account.scoped)
    @relation = relation
  end

  def find_each(&block)
    @relation.
      where(plan: nil, invites_count: 0).
      find_each(&block)
  end
end

```

可以利用這樣的方式寄信

``` ruby

AbandonedTrialQuery.new.find_each do |account|
  account.send_offer_for_support
end

```

還可以這樣用

``` ruby

old_accounts = Account.where("created_at < ?", 1.month.ago)
old_abandoned_trials = AbandonedTrialQuery.new(old_accounts)

```

## My Example

`before`

``` ruby

   registrants = Registrant.where(:paid_at => nil).where(["cancelled_at = ? ", nil ]).where("created_at <= ?", Time.now - 3.days)

    registrants.each do |registrant|
      registrant.cancelled_by_system_due_to_not_paid!
    end
```

`after`

``` ruby

class NeedCancelledRegistrant
  def initialize(relation = Registrant.scoped)
    @relation = relation
  end

  def find_each(&block)
    @relation.
      not_paid.not_cancelled.register_days_ago(3).
      find_each(&block)
  end
end

    NeedCancelledRegistrant.new.find_each do |r|
      r.cancelled_by_system_due_to_not_paid!
    end
```
