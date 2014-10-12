---
layout: post

title: "在Rails中使用Hstore"
cover_image: rails.jpg
tags:
- programming
- rails
- database
- postgresql
- ruby

excerpt: hstore是PostgreSQL中的一种数据类型，可以索引键值对，本文意说明了如何在Rails中使用hstore

author:
  name: 陈小紫 Cassius Chen
  twitter: cassiuschen
  bio: Ruby工程师，摄影设计爱好者.
  image: lg.png
---

Rails 4默认支持`hstore`，所以用起来得心应手。Rails4之前版本需要进行一些设置才可完成。为了支持hstore，PostgreSQL最好是9.0+版本。

####一、开启

对于Rails3.x，首先添加Gem:

{% highlight ruby %}
gem 'activerecord-postgres-hstore'
{% endhighlight %}

生成开启hstore的Migration

{% highlight sh %}
rails g hstore:setup
{% endhighlight %}

Rails4可以直接写一个新的migration，如下：

{% highlight ruby %}
class SetUpHstore < ActiveRecord::Migration
  def change
      enable_extension "hstore"
  end
end
{% endhighlight %}

运行

{% highlight sh %}
rake db:migrate
{% endhighlight %}

这样在写新的Migration的时候就可以用hstore格式了。

####二、配置

本例中我新建了一个Migration：

{% highlight ruby %}
class CreateNotices < ActiveRecord::Migration
  def change
    create_table :notices do |t|
      t.integer :page_id,	:primary_key => true
      t.string :content
      t.string :title
      t.hstore :extra

      t.timestamps
    end
  end
end
{% endhighlight %}

对于Rails3.x，在相应的Model中，写到：

{% highlight ruby %}
class Notice < ActiveRecord::Base
  serialize :extra, ActiveRecord::Coders::Hstore
end
{% endhighlight %}

Rails 4 则不用做任何改动。

之后新建这个Model的时候就可以写：

{% highlight ruby %}
notice = Notice.new( page_id = id )
notice.extra = {
  resource: "remote"
}
notice.save
{% endhighlight %}

这样就可以使用了。

####三、调用

如本例中的信息，调用时书写如下：

{% highlight ruby %}
n = Notice.find(id)
n.extra["resource"] #=> "remote"
{% endhighlight %}
