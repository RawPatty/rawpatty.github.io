---
layout: post
title:  "Github.io Blog Quickstart for Windows 10"
date:   2022-12-03 08:47:29 +0000
---
Having written a few blog posts in the past on my medium blog, I was inspired to move to blogging on Github pages like a "real" developer now that I've gotten comfortable writing posts.

As someone with an Infrastructure background, it was a bit daunting to get started, so part of this was also for my personal learning - I also wanted to start centralising my code and blog posts instead of having to create and link gists in Github as I currently do on Medium

As it took me quite a bit of time to set this all up (best part of a weekend), I thought it appropriate to make the first blog post about how I got the blog started.
I followed the [quickstart-guide-on-Github], so most of the content will be similar, but with a few twists to accomodate my own setup.

I run Windows 10, so to use the linux-based commands I ended up setting up an Ubuntu Dev container in my VSCode, guide here: [VS-Code-Container-Guide]

The DockerFile image I used is modified from [Docker-Image-Source]

I first created a repository in Github named `username.github.io`
I installed Ruby on my container with `sudo apt-get install ruby-full build-essential zlib1g-dev`
I installed Bundle on my container with `sudo apt-get install bundle`

I setup my Ruby gem environment variables first before installing
{% highlight ruby %}
export GEM_HOME="$HOME/gems
export PATH="$HOME/gems/bin:$PATH"
gem install jekyll bundler
{% endhighlight %}

I switched into my `docs` directory with cd commands and ran `jekyll new --skip-bundle`
which gave me the output `New jekyll site installed in /home/john/workspace/rawpatty.github.io/docs`

Finally, I modified the configuration GemFile with `vi GemFile` (requires vi to be installed)

First I commented out `gem "jekyll", "~> 4.3.1"`

Then I modified the line starting `gem "github-pages"` to 

`gem "github-pages", "~> 227", group: :jekyll_plugins`

Then ran `bundle install`
 
And that was it!

New posts are written in markdown under the `_posts` folder and follow the naming format
`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

A part that had confused me for a while was how to locate the actual post you wrote, but the URL is generated following your page name with the dates as dividers (I will update this as I learn more)
eg.
`https://rawpatty.github.io/jekyll/update/2022/12/03/First-page-example.html`

Things I want to learn next are how to add pictures, categorise articles into their own repos, and how to simplify the URL so it removes the `jekyll/update` path

Stay Tuned (and hopefully I don't procrastinate this)

Update: Turns out the URL is generated from the categories, so as above I would have two categories of jekyll and update - by removing the categories section, I was able to make the blog post directly link appear without categories appearing in the URL.

<img src="{{site.baseurl | prepend: site.url}}../images/2022/removeCategories.png" alt="zigzag" />

Testing 


![test2](../images/2022/removeCategories.png)

Below is some helpful source doco as part of the default page but I'll leave in for interest:


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
[quickstart-guide-on-Github]: https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll
[VS-Code-Container-Guide]: https://code.visualstudio.com/docs/devcontainers/containers
[Docker-Image-Source]: https://github.com/microsoft/vscode-dev-containers/blob/v0.122.1/containers/ubuntu/.devcontainer/base.Dockerfile