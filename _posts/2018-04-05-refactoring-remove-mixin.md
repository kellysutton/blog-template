---
layout: post
title: "Refactoring: Remove Mixin"
date: 2018-04-05T07:41:00-07:00
excerpt: Why would we want to remove mixins from our code and how can we?
---

In Object-Oriented Programming, [mixins](https://en.wikipedia.org/wiki/Mixin) are a powerful concept to elminate duplication in code. They allow you to extract common behavior into a shared module.

Over time, each programming language community seems to embrace mixins and then eschew them. [Mixins are now no longer recommended within the React ecosystem.](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html) ActiveSupport::Concern was [introduced into Rails 3.0 in 2010, which is the Rails flavor of a mixin](http://api.rubyonrails.org/v5.1/classes/ActiveSupport/Concern.html). At Gusto, we’ve become wary of introducing any new Concerns to our code base.

This post covers why mixins can be dangerous to a large code base and proposes a refactoring process for methodically removing them.

## Considered Harmful?

When disliking a certain thing, software engineers often attempt to describe something as objectively harmful. It’s sometimes difficult to separate the emotion from the rationale.

[Dan Abramov](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html) does a good job to highlight that mixins do the following 3 things to a code base:

0. Mixins introduce implicit dependencies
0. Mixins cause name clashes
0. Mixins cause snowballing complexity

Mixins indeed do these three things. (1) and (2) in particular can be difficult to deal with in languages without static analysis. All three of these points though I believe are symptoms of another higher-order problem (see what I did there?).

The problem in particular is that mixins destroy the dependency graph of our applications unlike any other construct. By muddying the implicit parts of our dependency graphs, our applications snowball in complexity.

Let’s see how that looks pictorially.

## Mixins, Concerns, and Grenades

Let’s say we have a simple blogging system with `Post`, `Author`, and `Comment` classes. Let’s assume that both `Author` and `Comment` belong to `Post`.

<figure>
  {% image_tag /images/refactoring-remove-mixin-simple-dag.png %}
  <figcaption>Our simple application</figcaption>
</figure>

Now let’s take a look at the Ruby/Rails code associated with each of these classes:

```ruby
class Post
  has_many :authors
  has_many :comments
end

class Author
  belongs_to :post
end

class Comment
  belongs_to :post
end
```

Now with any system, we know that this pristine state won’t last for long. Let’s assume that we need to add a summary to each class:

```ruby
class Post
  # ...

  def summary
    [title, published_at].join(' - ')
  end
end

class Author
  # ...

  def summary
    [first_name, last_name].join(' ')
  end
end

class Comment
  # ...

  def summary
    body[0...100] + '…'
  end
end
```

Let’s assume this common `#summary` method is used throughout the application.[^1]

We think we know of a way to DRY up this code by moving all definitions of `#summary` into a mixin. This example is a bit sinister because each `#summary`’s definition is particular to that class. Nonetheless, these are things that I have seen in the wild before and done myself.

So we make the change:

```ruby
module Summarizable
  extend ActiveSupport::Concern

  included do
    def summary
      case self
      when Post
        [title, published_at].join(' - ')
      when Author
        [first_name, last_name].join(' ')
      when Comment
        body[0...100] + '…'
      end
    end
  end
end

class Post
  include Summarizable
  # ...
end

class Author
  include Summarizable
  # ...
end

class Comment
  include Summarizable
  # ...
end
```

While we have technically reduced the total lines of code in the models themselves, we have done some serious damage to our application structure. Let’s take a look:

<figure>
  {% image_tag /images/refactoring-remove-mixin-with-mixin.png %}
  <figcaption>Our application with each object referring to the mixin</figcaption>
</figure>

But it gets worse than this. We are not just adding a dependency, but we are also adding a dependency onto our neighboring classes as well.

<figure>
  {% image_tag /images/refactoring-remove-mixin-with-mixin-dotted-lines.png %}
  <figcaption>Because of the way the mixin is written, each object subtly depends on every other object!</figcaption>
</figure>

Our dependency graph has become a dependency spider web. Because the mixin knows what it’s being mixed in to, each object now depends on each other. We have a big circular dependency. This will make each of these objects more difficult to change because they depend on each other.

Oftentimes the dependency is not quite as explicit as a direct class reference or a direct `import` or `require`. Instead, the circular dependencies we weave are conceptual or implicit circular dependencies. Your code may not directly reference different classes, but it still may assume they exist.

Although many Rails developers have a love/hate relationship with the [autoloading mechanism](http://guides.rubyonrails.org/autoloading_and_reloading_constants.html), it is a good litmus test for bad patterns. Things that trigger too many loads might be a bad pattern. In this case, asking for `Comment#summary` will trigger a load on the `Post` and `Author` classes in development mode.

The more places this is mixed in to, the tighter these classes get coupled together. Rather than by decreasing coupling by extracting code into a different file, we increase coupling. Even in cases less contrived than this, the problem may still exist or develop over time.

## What can be done instead?

In React, we reach for composition through higher-order components. In Ruby, we reach for composable stateless service objects.

Let's take a look at the code in this example:

```ruby
class Summarizer
  def self.summary(object)
    case object
    when Post
      [object.title, object.published_at].join(' - ')
    when Author
      [object.first_name, object.last_name].join(' ')
    when Comment
      object.body[0...100] + '…'
    end
  end
end

class Post
  # ...
end

class Author
  # ...
end

class Comment
  # ...
end
```

At first glance, this might not look much different than our mixin based approach. Furthermore, usage of this feels more awkward. Rather than a nice `post.summary` in our code, we now have to do `Summarizer.summary(post)`. Surely I couldn’t be advocating for such a doubling of characters just to generate a summary?

This new approach is objectively less tangled than the mixin-based approach. [It is simpler but it is less easy.](https://www.infoq.com/presentations/Simple-Made-Easy) (If you don’t understand the distinction, please watch the linked talk.) Let’s take a look at our dependency graph:

<figure>
  {% image_tag /images/refactoring-remove-mixin-service-class.png %}
  <figcaption>A service class or composition approach does not change the number of nodes in our graph, but simplifies our relationships</figcaption>
</figure>

This structure is much cleaner and keeps our graph acyclic. Should we no longer need to generate summaries, we can easily remove the service class without wondering if anyone else relies on `Summarizer`. (Or if other code used the `Summarizer`, that would be a simple Find operation.)

Now that we have the start and the finish states, let’s have a look at the individual steps to this refactoring.

## Individual Steps

1. Identify mixin to convert.
2. Identify the test coverage of the mixin. If no tests, write some. Write these tests based on the mixin’s behavior from the object that it’s mixed in to. For example, `Post#summary` in the above example.
3. Create empty service class to move the behavior into.
4. Copy a method or case from the mixin into the new service class.
5. Replace existing usage of mixin behavior with service class behavior.
6. Using the tests, assert that the behavior has not changed.
7. “Push up” the behavior of the mixin into the classes themselves.
8. Repeat Steps 4 - 7 until all code is converted to using the service class.
9. Remove the mixin.
10. Replace usages of instance method with new service class.
11. Delete dead method in original class.
12. Repeat Steps 10 - 11 until only the service class is used.

Quite a few steps. Let’s break them down one by one. For this part, we’ll use the most barebones of code. After each step, run your tests!

### 1. Identify mixin to convert

Usually this is easy, let’s use the following code:

```ruby
class Post
  include Summarizable
end

module
  extend ActiveSupport::Concern

  included do
    def summary
      case self
      when Post
        [title, published_at].join(' - ')
      end
    end
  end
end
```

Mixin: identified.

### 2. Identify the test coverage of the mixin.

Let’s assume this mixin has the following tests (written in RSpec):

```ruby
require 'rails_helper'

RSpec.describe Post do
  describe '#summary' do
    subject { post.summary }

    let(:post) do
      Post.new(
        title: "You won't believe",
        published_at: Date.new(2018, 4, 1)
      )
    end

    it { is_expected.to eq("You won't believe - 2018-01-01") }
  end
end
```

### 3. Create empty service class to move the behavior into.

We want to give a home for where we’ll be moving the code. Here’s how the code might look at this step:

```ruby
class Summarizer
end

class Post
  include Summarizable
end

module
  extend ActiveSupport::Concern

  included do
    def summary
      case self
      when Post
        [title, published_at].join(' - ')
      end
    end
  end
end
```

### 4. Copy a method or case from the mixin into the new service class.

We’ll have some duplicated code at this step, but that’s okay.

```ruby
class Summarizer
  def self.summarize(object)
    case object
    when Post
      [object.title, object.published_at].join(' - ')
    end
  end
end

class Post
  include Summarizable
end

module
  extend ActiveSupport::Concern

  included do
    def summary
      case self
      when Post
        [title, published_at].join(' - ')
      end
    end
  end
end
```

### 5. Replace existing usage of mixin behavior with service class behavior.

Rather than immediately removing the method, we want to make sure we copied the behavior over correctly:

```ruby
class Summarizer
  def self.summarize(object)
    case object
    when Post
      [object.title, object.published_at].join(' - ')
    end
  end
end

class Post
  include Summarizable
end

module
  extend ActiveSupport::Concern

  included do
    def summary
      case self
      when Post
        Summarizer.summarize(self)
      end
    end
  end
end
```

### 6. Using the tests, assert that the behavior has not changed.

Yep. We should be doing this constantly. We may also want to take this time to start filling out the specs for our new `Summarizer`.

### 7. “Push up” the behavior of the mixin into the classes themselves.

Now it’s time to remove code from the mixin as we most it into `Post`:

```ruby
class Summarizer
  def self.summarize(object)
    case object
    when Post
      [object.title, object.published_at].join(' - ')
    end
  end
end

class Post
  include Summarizable

  def summary
    Summarizer.summarize(self)
  end
end

module
  extend ActiveSupport::Concern

  included do
    def summary
      case self
      end
    end
  end
end
```


### 8. Repeat Steps 4 - 7 until all code is converted to using the service class.

Looks like we’re done in this example. Nothing more to do here.


### 9. Remove the mixin.

Nothing easier than safely deleting code

```ruby
class Summarizer
  def self.summarize(object)
    case object
    when Post
      [object.title, object.published_at].join(' - ')
    end
  end
end

class Post
  def summary
    Summarizer.summarize(self)
  end
end
```

### 10. Replace usages of instance method with new service class.

Let’s assume we have a template or an email somewhere using `post.summary`. We’d replace that with `Summarizer.summary(post)`.

### 11. Delete dead method in original class.

Now that `Post#summary` is no longer used, we remove it.

```ruby
class Summarizer
  def self.summarize(object)
    case object
    when Post
      [object.title, object.published_at].join(' - ')
    end
  end
end

class Post
end
```

### 12. Repeat Steps 10 - 11 until only the service class is used.

And we’re done!

## Conclusion

Hopefully this refactoring technique is useful and cleans up the dependency graphs drawn by your applications. If you have any feedback, please don’t hesitate to get in touch [via Twitter](https://twitter.com/kellysutton) or [email](mailto:michael.k.sutton@gmail.com).

There is some further reading that might help you on your quest:

- [Mixins: A refactoring anti-pattern](http://blog.steveklabnik.com/posts/2012-05-07-mixins--a-refactoring-anti-pattern) by Steven Klabnik
- [Mixins Are Not Always a Refactoring Anti-pattern](http://mattbriggs.net/blog/2012/05/07/mixins-are-not-a-refactoring-anti-pattern/) by Matt Briggs
- [Why Are Mixins Considered Harmful?](http://raganwald.com/2016/07/16/why-are-mixins-considered-harmful.html) by Reginald Braithwaite
- [Mixins Considered Harmful](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html)
- [Mixin on Wikipedia](https://en.wikipedia.org/wiki/Mixin)
- [Nothing is Something](https://www.youtube.com/watch?v=29MAL8pJImQ) by Sandi Metz

---

Special thanks to [Sihui Huang](http://www.sihui.io/), [Omri Ben Shitrit](https://www.linkedin.com/in/omri-ben-shitrit-2464293/), and [Justin Duke](https://twitter.com/justinmduke) for providing feedback on early drafts of this post.

{% include newsletter.html %}

[^1]: This type of thing is very much a presentation concern and doesn’t belong at the model layer, but let’s forget about that for now.
