# Extract Service Object

<http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/>

## 適合場景

* 複雜的 action
* action 裡面牽涉到多個 model （ 如 e-commerce 的結帳，牽扯到訂單, 使用者, 明細的修改 ) 
* action 裡面牽扯到外部 service ( 例如 po 到 twitter 上)
* action 的不是這個 model 核心業務的一部分（如清除舊資料...)
* 有很多方式可以執行這樣的 action (例如 驗證帳號密碼). 這是四人幫裡的 [Strategy Patten](http://en.wikipedia.org/wiki/Strategy_pattern)

``` ruby UserAuthenticator

class UserAuthenticator
  def initialize(user)
    @user = user
  end

  def authenticate(unencrypted_password)
    return false unless @user

    if BCrypt::Password.new(@user.password_digest) == unencrypted_password
      @user
    else
      false
    end
  end
end

```


``` ruby SessionsController

class SessionsController < ApplicationController
  def create
    user = User.where(email: params[:email]).first

    if UserAuthenticator.new(user).authenticate(params[:password])
      self.current_user = user
      redirect_to dashboard_path
    else
      flash[:alert] = "Login failed."
      render "new"
    end
  end
end

```