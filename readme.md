## JK Jung's blog

[https://jkjung-avt.github.io/](https://jkjung-avt.github.io/)

My blog is deeply inspired by [Andrej Karpathy's work](http://karpathy.github.io/). I write this blog with [Jekyll](http://jekyllrb.com/), and I have forked [adueck's cayman-blog](https://github.com/adueck/cayman-blog) theme.

My blog is hosted on [GitHub Pages](https://pages.github.com/). Meanwhile I've also set up Jekyll on my own Ubuntu 14.04 x64 PC for previewing the pages/posts before committing them to GitHub. Below I briefly list the procedure to get Jekyll working locally. For more details, you can refer to the [official Jekyll installation documentation](https://jekyllrb.com/docs/installation/).

```shell
 ### Installing ruby, version 2.3.3
 $ sudo apt-get install libssl-dev
 $ mkdir -p ~/src
 $ cd ~/src
 $ wget https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.3.tar.gz
 $ tar xzvf ruby-2.3.3.tar.gz
 $ cd ruby-2.3.3/
 $ ./configure
 $ make
 $ sudo make install
 $ ruby --version

 ### Updating gem to the latest version, 2.6.10
 $ sudo gem update --system
 $ gem --version

 ### Installing bundler, version 1.14.3
 $ sudo gem install bundler
 $ bundle --version

 ### I don't explicitly install Jekyll here. Instead, I rely on 'bundle install' in the target direcoty to install the proper version of Jekyll for me.
```

After checking out jkjung-avt.github.io repository onto my local PC for the first time, I'd run `bundle install` once to make sure Jekyll (3.3.1) and all required gems for cayman-blog theme are installed properly.  

```shell
 $ git clone https://github.com/jkjung-avt/jkjung-avt.github.io.git
 $ cd jkjung-avt.github.io/
 $ bundle install
 
 ### Then run the following command to preview the pages/posts
 ### locally @ http://localhost:4000
 $ bundle exec jekyll serve
```

In addition, I have integrated Disqus onto my blog by referencing [Brendan A R Sechter's Development Blog](http://sgeos.github.io/jekyll/disqus/2016/02/14/adding-disqus-to-a-jekyll-blog.html). I've also integrated Google Analytics to track website traffic. My Disqus URL and shortname settings are located in [_include/disqus.html](https://github.com/jkjung-avt/jkjung-avt.github.io/blob/master/_includes/disqus.html), while my Google Analytics tracking code in [_include/analytics.html](https://github.com/jkjung-avt/jkjung-avt.github.io/blob/master/_includes/analytics.html).

**JK Jung**

<jkjung13@gmail.com>

