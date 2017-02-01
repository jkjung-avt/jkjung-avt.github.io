## JK Jung's blog

My blog is deeply inspired by [Andrej Karpathy's work](http://karpathy.github.io/). I write this blog with [Jekyll](http://jekyllrb.com/), and I have forked [adueck's caymen-blog](https://github.com/adueck/cayman-blog) theme.

My blog is hosted on [GitHub Pages](https://pages.github.com/). Meanwhile I've also set up Jekyll on my own Ubuntu 14.04 x64 PC for previewing the pages/posts before committing them to GitHub. Below I briefly list the procedure to get Jekyll working locally. For more details, you can refer to the [official Jekyll installation documentation](https://jekyllrb.com/docs/installation/).

```shell
 $ sudo apt-get install libssl-dev
 $ mkdir -p ~/src
 $ cd ~/src
 $ wget https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.0.tar.gz
 $ tar xzvf ruby-2.4.0.tar.gz
 $ cd ruby-2.4.0/
 $ ./configure
 $ make
 $ sudo make install
 $ ruby --version
 $ cd ~/src
 $ wget https://rubygems.org/rubygems/rubygems-2.6.8.tgz
 $ tar xzvf rubygems-2.6.8.tgz
 $ cd rubygems-2.6.8/
 $ sudo ruby setup.rb
 $ gem --version
 $ sudo gem install jekyll
 $ jekyll --version
 $ sudo gem install bundler
 $ bundle --version
```

And after checking out jkjung-avt.github.io repository onto my local PC for the first time, I'd run 'bundle install' once to make sure all required gems for caymen-blog theme are installed properly.

```shell
 $ git clone https://github.com/jkjung-avt/jkjung-avt.github.io.git
 $ cd jkjung-avt.github.io/
 $ bundle install
 
 ### Then run the following command to preview the pages/posts
 ### locally @ http://localhost:4000
 $ bundle exec jekyll serve
```

Moreover I referenced [Brendan A R Sechter's Development Blog](http://sgeos.github.io/jekyll/disqus/2016/02/14/adding-disqus-to-a-jekyll-blog.html) for how to integrate Disqus onto my Jekyll blogs.

<jkjung13@gmail.com>

