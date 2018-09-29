title: My First Post Using Hexo
date: 2015-01-01 19:19:39
tags:
- firstpost
- hexo
comments: false
---
I started working as a software developer for a large company about a month ago.  Up until that point, I was working in a different profession, and programming was mostly just my hobby.  I've programmed in a lot of different areas, both for work and for fun: PLC programming, HMI programming, MATLAB, and python.  My work programming has mostly been related to machine software, and my hobbyist programming has mostly been web development. Now I'm working with JavaScript, although it's unclear how long that will be true.  I am not formally trained in computer science.

I am starting this blog in part as an effort to chronicle my learning process as a software developer.  As an aspiring developer, I know there are many others who are not trained in computer science or computer engineering and want to break into the field.  Maybe I can help them somehow.

I also just wanted to get writing.  I wanted an outlet where I can write about things that interest me.  What are my interests besides programming?  Technology, gadgets, science and engineering, music, health and nutrition, to name just a few.  Although this blog will mostly be about programming and tech, I'm not going to necessarily narrow the scope to just that.  Who knows?

Since I'm going to be working with JavaScript a lot, I decided to use a JavaScript-based blogging framework called [Hexo](http://hexo.io).  Hexo is powered by Node.js.  It's simple and elegant, and it uses [Markdown](http://daringfireball.net/projects/markdown/syntax) for blog posts.  Installation and setup was easy in Ubuntu:

	$ npm install -g hexo
	$ hexo init my-blog
	$ cd my-blog
	$ npm install

Now I have a blog located in the folder my-blog.  Then to create a new blog post, I just do

	$ hexo new my-new-blog-post

This generates a new markdown file with the title in the filename.  I've configured the filename format for new posts so that it also includes the date, like so:

``` yaml
new_post_name: :year-:month-:day-:title.md # File name of new posts
```

This is located in the site configuration file, _config.yml, along with a variety of other options, such as the site title, URL, language, and server options.  I really like the default theme that comes with hexo.  In any case, there are a variety of other themes available at [https://github.com/hexojs/hexo/wiki/themes](https://github.com/hexojs/hexo/wiki/themes).  One thing I noticed is that the RSS feed doesn't seem to work right out of the box.  To get it working, you just need to install the feed generator plugin, like so:

	$ npm install hexo-generator-feed --save

Then you can add it to your configuration file:

``` yaml
# RSS Feed
feed:
  type: atom
  path: atom.xml
  limit: 20
```

That's about it for now.  I will probably update this post as I learn more about hexo in the future.