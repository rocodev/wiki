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