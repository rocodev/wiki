# Replace conditional with Null Object

## 適合場景

有以下條件式：

* nil?
* present?
* try? 

出現的時候。


## Example

### 空字串

`app/models/question.rb`


``` ruby
def most_recent_answer_text 
  answers.most_recent.try(:text) || Answer::MISSING_TEXT
end

```

### 物件可能為空

` app/models/answer.rb`

``` ruby

def self.most_recent
  order(:created_at).last
end

```

可以改寫如以下

``` ruby

# app/models/answer.rb
class Answer < ActiveRecord::Base
  include ActiveModel::ForbiddenAttributesProtection
  belongs_to :completion
  belongs_to :question
  validates :text, presence: true

  def self.for_user(user)
    joins(:completion).where(completions: { user_id: user.id }).last || NullAnswer.new
  end

  def self.most_recent 
    order(:created_at).last || NullAnswer.new
  end 
end


# app/models/null_answer.rb
class NullAnswer
  def text
    "No response"
  end 
end

```


## My example

`before`

``` ruby
# app/models/course.rb
 def area_name
   "#{course.area.try(:name)}" || "尚未設定開課地區"
 end
```

`after`

``` ruby
# app/models/course.rb
  def area_name
    course_area = area  || NullArea.new
    course_area.name
  end

# app/models/null_area.rb

class NullArea
  def name
    "尚未設定開課地區"
  end
end

```

