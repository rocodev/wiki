# Extract Policy Objects

## 適合場景

有時候複雜的 read operations 不適合放在 Model 裡。這時候它們應該要放在屬於自己的 objects 裡。
這時候可以設計 Policy Objects


``` ruby

class ActiveUserPolicy
  def initialize(user)
    @user = user
  end

  def active?
    @user.email_confirmed? &&
    @user.last_login_at > 14.days.ago
  end
end

```
