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


## My Example

### 第一個實例： 通知相關人士

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


### 第二個實例 :  報名付款

`before`

`app/controllers/registrations_controller.rb`

``` ruby

if payment.success?
  current_user.pay_registration_with_paypal(amount, registration_id)
end

```

`app/models/user.rb`

``` ruby
def pay_registration_with_paypal(amount, registration_id)
  registration = Registrant.find(registration_id)
  registration.pay!(amount, "paypal")
end
```

`app/models/registrant.rb`

``` ruby

  def pay!(amount, payment_type)
    r              = self
    r.paid_amount  = amount
    r.payment_type = payment_type
    r.paid_at      = Time.now
    r.save
    r.generate_invoice(amount, payment_type)
    r.notify_teacher_payment_complete
    r.notify_student_payment_complete
  end

  def notify_teacher_payment_complete
    @student = self.user
    @teacher = self.course.user
    @course  = self.course
    @amount  = self.paid_amount
    @invoice = self.invoice
    PaymentMailer.delay.notify_teacher_payment_complete(@student, @teacher, @course, @amount, @invoice)
  end

  def notify_student_payment_complete
    @student = self.user
    @teacher = self.course.user
    @course  = self.course
    @amount  = self.paid_amount
    @invoice = self.invoice
    PaymentMailer.delay.notify_student_payment_complete(@student, @teacher, @course, @amount, @invoice)
  end


  def generate_invoice(amount, payment_type)
    r                    = self
    invoice              = Invoice.new
    invoice.resource     = r
    invoice.amount       = amount
    invoice.payment_type = payment_type
    invoice.paid_at      = Time.now
    invoice.user         = r.user
    invoice.save
  end
```

`after`

`app/controllers/registrations_controller.rb`

``` ruby
if payment.success?
    PayRegistrant.new(registration_id, amount, "paypal" ).pay!
end
```


`app/services/pay_registrant.rb`

``` ruby
class PayRegistrant

  def initialize(registrant_id, amount, payment_type)
    @registrant = Registrant.find(registrant_id)
    @amount = amount
    @payment_type = payment_type
  end

  def pay!
    update_registrant_paid_at
    generate_invoice_for_registrant
    notify_teacher_payment_complete
    notify_student_payment_complete
  end

  def update_registrant_paid_at
    @registrant.paid_amount  = @amount
    @registrant.payment_type = @payment_type
    @registrant.paid_at      = Time.now
    @registrant.save
  end

  def generate_invoice_for_registrant
    invoice = @registrant.build_invoice
    invoice.amount       = @amount
    invoice.payment_type = @payment_type
    invoice.paid_at      = Time.now
    invoice.user         = @registrant.user
    invoice.save
  end

  def notify_teacher_payment_complete
    PaymentMailer.delay.notify_teacher_payment_complete(@registrant)
  end

  def notify_student_payment_complete
    PaymentMailer.delay.notify_student_payment_complete(@registrant)
  end
end
```


