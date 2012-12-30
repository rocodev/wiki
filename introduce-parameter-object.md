# Introduce Parameter Object

## 適用場景

* parameter 過多
* parameters 彼此適合收集在一起
* 封裝有關的 parameters 的行為


``` ruby

class Mailer < ActionMailer::Base
  default from: "from@example.com"
  def completion_notification(first_name, last_name, email)
    @first_name = first_name
    @last_name = last_name
    mail(
      to: email,
      subject: ’Thank you for completing the survey’
)
  end 
end

```

``` ruby

class Mailer < ActionMailer::Base
  default from: "from@example.com"
  def completion_notification(recipient)
    @recipient = recipient
    mail(
      to: recipient.email,
      subject: ’Thank you for completing the survey’
)
  end 
end

class Recipient
  def initialize(first_name, last_name, email)
    @first_name = first_name
    @last_name = last_name
    @email = email
  end
end

```