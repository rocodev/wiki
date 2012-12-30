# Extract Decorators

<http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/>

## 場景

跟 Service Object 不一樣的是，Decorator 只基於相同責任的原有物件與界面做擴充。

``` ruby

class FacebookCommentNotifier
  def initialize(comment)
    @comment = comment
  end

  def save
    @comment.save && post_to_wall
  end

private

  def post_to_wall
    Facebook.post(title: @comment.title, user: @comment.author)
  end
end

```

``` ruby

class CommentsController < ApplicationController
  def create
    @comment = FacebookCommentNotifier.new(Comment.new(params[:comment]))

    if @comment.save
      redirect_to blog_path, notice: "Your comment was posted."
    else
      render "new"
    end
  end
end

```

