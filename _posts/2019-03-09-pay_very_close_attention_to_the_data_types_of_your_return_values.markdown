---
layout: post
title:      "Pay VERY Close Attention to the Data Types of Your Return Values"
date:       2019-03-09 10:30:16 -0500
permalink:  pay_very_close_attention_to_the_data_types_of_your_return_values
---



Last week, I completed the ORMs and ActiveRecord portion of the Flatiron curriculum. It was a heavy subject with a lot of material, so rather than continue with the next session I decided to take a break from the curriculum and practice what I had learned in that section. I created a [Blog CLI](https://github.com/tgray017/blog_cli) with ActiveRecord models and migrations, and in order to test it I tried to seed my database with instances of `User`s, `Post`s, and `Comment`s.

A comment belongs to a commentor, which I set up to be an alias of the `User` class, and a comment belongs to a post. I have these associations set up in my model using the `:belongs_to` metaprogramming concept. I created a couple of users, myself and my wife, and created a couple of posts, one of which was about our dog, Wilson (I filled the post with some garbage I got from [Hipster Ipsum](https://hipsum.co/)).

First, I identified the post that I wanted my wife's user to comment on:
```
[2] pry(Seed)> commented_post = Post.where(:title => "Wilson is the best dog in the whole world")
=> [#<Post:0x007f9a01423cc0
  id: 2,
  title: "Wilson is the best dog in the whole world",
  content:
   "Air plant bitters hot chicken, edison bulb XOXO iceland polaroid bushwick retro try-hard. Ennui you probably haven't heard of them tattooed austin bespoke retro, pinterest tacos narwhal actually deep v stumptown knausgaard swag. 90's biodiesel chicharrones edison bulb flannel. Gochujang ramps cornhole meh aesthetic, dreamcatcher church-key kombucha celiac cardigan tumblr lomo. Selvage street art organic 3 wolf moon, paleo lumbersexual neutra vice humblebrag disrupt cornhole kitsch church-key.",
  author_id: 1>]
```

Unfortunately for me, I didn't notice the brackets surrounding the return value, which indicated an array. This miss on my part came back to bite me in the a$$. But at that point, I simply pushed forward, unaware that the return value was _not_ an instance of the `Post`class.

I then created the comment using the `#build` method provided by ActiveRecord:
```
1] pry(Seed)> wilson_comment = victoria.comments.build(:content => "Yes he is!")
=> #<Comment:0x007f9a012c9b68 id: nil, content: "Yes he is!", post_id: nil, commentor_id: nil>
```

So when I tried to associate the comment with the post, I got the following error:

```
Error:
[3] pry(Seed)> wilson_comment.post = commented_post
ActiveRecord::AssociationTypeMismatch: Post(#70149712581500) expected, got #<ActiveRecord::Relation [#<Post id: 2, title: "Wilson is the best dog in the whole world", content: "Air plant bitters hot chicken, edison bulb XOXO ic...", author_id: 1>]> which is an instance of Post::ActiveRecord_Relation(#70149712580500)
from /Users/thomas.gray/.rvm/gems/ruby-2.3.1/ruby/2.3.0/gems/activerecord-5.2.2/lib/active_record/associations/association.rb:246:in `raise_on_type_mismatch!'
```

This confused the hell out of me, because it looked like what it was expecting was exactly what it was getting (the instance `Post(#70149712581500))`. It took me 4 hours of googling to figure out it wasn't a bug with ActiveRecord, and when I read through the error for the millionth time I finally realized that the return value of `commented_post = Post.where(:title => "Wilson is the best dog in the whole world")` was an _array_ of `Post` instances--an array that only had one item. All I had to do was change this to `commented_post = Post.where(:title => "Wilson is the best dog in the whole world").first` and I finally succeeded in seeding my development database. 

In short, if you want to save yourself some pain, pay VERY close attention to the data types of your return values.



