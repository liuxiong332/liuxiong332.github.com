---
layout: post
title:  "Hello World"
date:   2015-01-12 10:25:49
categories: Hello World
---

# The largest heading (an <h1> tag)
## The second largest heading (an <h2> tag)
### The third largest heading (an <h3> tag)

> Pardon my french, **Block Quote**

do_this_and_do_that_and_another_thing.

[Visit GitHub!](www.github.com)

https://www.google.com

*This text will be italic*
**This text will be bold**
~~Mistaken text. strike through~~

Here's an idea: why don't we take `SuperiorProject` and turn it into `**Reasonable**Project`.

* Item
* Item
* Item

1. Item 1
2. Item 2
3. Item 3

1. Item 1
  1. A corollary to the above item.
  2. Yet another point to consider.
2. Item 2
  * A corollary that does not need to be ordered.
    * This is indented four spaces, because it's two spaces further than the item above.
    * You might want to consider making a new list.
3. Item 3

First Header | Second Header
-------------|---------------
Content Cell | Content Cell
Content Cell2| Content Cell2

| Name | Description          |
| ------------- | ----------- |
| Help      | Display the help window.|
| Close     | Closes a window     |

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |

{% highlight javascript%}
function ThreadPaneSelectionChanged()
{
	GetThreadTree().view.selectionChanged();
}
{% endhighlight %}

{% highlight html %}
<html>
  <head><title>Title!</title></head>
  <body>
    <p id="foo">Hello, World!</p>
    <script type="text/javascript">var a = 1;</script>
    <style type="text/css">#foo { font-weight: bold; }</style>
  </body>
</html>
{% endhighlight %}

{% highlight ruby %}
class Greeter
  def initialize(name="World")
    @name = name
  end

  def say_hi
    puts "Hi #{@name}!"
  end
end
{% endhighlight %}

```ruby
require 'abstract_unit'

class TestAutoloadModule < ActiveSupport::TestCase
  include ActiveSupport::Testing::Isolation

  module ::Fixtures
    extend ActiveSupport::Autoload

    module Autoload
      extend ActiveSupport::Autoload
    end
  end

  def setup
    @some_class_path = File.expand_path("test/fixtures/autoload/some_class.rb")
    @another_class_path = File.expand_path("test/fixtures/autoload/another_class.rb")
  end

  test "the autoload module works like normal autoload" do
    module ::Fixtures::Autoload
      autoload :SomeClass, "fixtures/autoload/some_class"
    end

    assert_nothing_raised { ::Fixtures::Autoload::SomeClass }
  end

  test "when specifying an :eager constant it still works like normal autoload by default" do
    module ::Fixtures::Autoload
      eager_autoload do
        autoload :SomeClass, "fixtures/autoload/some_class"
      end
    end

    assert !$LOADED_FEATURES.include?(@some_class_path)
    assert_nothing_raised { ::Fixtures::Autoload::SomeClass }
  end

  test "the location of autoloaded constants defaults to :name.underscore" do
    module ::Fixtures::Autoload
      autoload :SomeClass
    end

    assert !$LOADED_FEATURES.include?(@some_class_path)
    assert_nothing_raised { ::Fixtures::Autoload::SomeClass }
  end

  test "the location of :eager autoloaded constants defaults to :name.underscore" do
    module ::Fixtures::Autoload
      eager_autoload do
        autoload :SomeClass
      end
    end

    assert !$LOADED_FEATURES.include?(@some_class_path)
    ::Fixtures::Autoload.eager_load!
    assert $LOADED_FEATURES.include?(@some_class_path)
    assert_nothing_raised { ::Fixtures::Autoload::SomeClass }
  end

  test "a directory for a block of autoloads can be specified" do
    module ::Fixtures
      autoload_under "autoload" do
        autoload :AnotherClass
      end
    end

    assert !$LOADED_FEATURES.include?(@another_class_path)
    assert_nothing_raised { ::Fixtures::AnotherClass }
  end

  test "a path for a block of autoloads can be specified" do
    module ::Fixtures
      autoload_at "fixtures/autoload/another_class" do
        autoload :AnotherClass
      end
    end

    assert !$LOADED_FEATURES.include?(@another_class_path)
    assert_nothing_raised { ::Fixtures::AnotherClass }
  end
end
```
