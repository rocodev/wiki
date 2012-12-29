# Extract Service Object

<http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/>

## 適合場景

* 複雜的 action
* action 裡面牽涉到多個 model （ 如 e-commerce 的結帳，牽扯到訂單, 使用者, 明細的修改 ) 
* action 裡面牽扯到外部 service ( 例如 po 到 twitter 上)
* action 的不是這個 model 核心業務的一部分（如清除舊資料...)
* 有很多方式可以執行這樣的 action (例如 驗證帳號密碼). 這是四人幫裡的 [Strategy Patten](http://en.wikipedia.org/wiki/Strategy_pattern)

`UserAuthenticator`

``` ruby 

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

`SessionsController`


``` ruby 

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


### My Example

`before`

`app/controllers/comments_controller.rb`

``` ruby
  def create
    @comment      = @resource.comments.new(params[:comment])
    @comment.user = current_user

    if @comment.save
       @comment.notify_commneters
      redirect_to(@resource, :notice => "已送出留言")
    else
      render :new
    end
  end
```

`app/models/comment.rb`

``` ruby

class Comment

  def notify_commneters
    teacher = resource.teacher
    commenter = user

    comment_related_users = []
    comment_related_users << resource.comment_users
    comment_related_users << teacher
    comment_related_users.uniq!
    comment_related_users.delete(commenter)

    comment_related_users.each do |related_user|
      CommentMailer.delay.comment_course_email(related_user, self)
    end
  end
end

```


`after`

`app/controllers/comments_controller.rb`

``` ruby
  def create
    @comment      = @resource.comments.new(params[:comment])
    @comment.user = current_user

    if @comment.save
      CommentNotifer.notify_related_users(@comment)
      redirect_to(@resource, :notice => "已送出留言")
    else
      render :new
    end
  end
```

`app/services/comment_notifier.rb`

``` ruby

class CommentNotifier

  def self.notify_related_users(comment)
    resource = comment.resource
    teacher = resource.teacher
    commenter = comment.user

    comment_related_users = []
    comment_related_users << resource.comment_users
    comment_related_users << teacher
    comment_related_users.uniq!
    comment_related_users.delete(commenter)

    
    comment_related_users.each do |related_user|
      CommentMailer.delay.comment_course_email(related_user, self)
    end
  end
end

```