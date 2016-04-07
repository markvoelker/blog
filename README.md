Future Relics
=============

*The rantings of some nutcase on the internet who knows most of the
technology he creates for a living will be obsolete long before he
retires...and not only continues doing it, but actually enjoys it.*

How to publish posts
--------------------

The site is generated using [Hugo](https://gohugo.io/).  You'll
need to install it on your local machine to do anything useful.

The workflow I'm using as of this writing is modeled after
[this one](https://gohugo.io/tutorials/github-pages-blog/).

To write a post, you'll need to clone the repo locally:

`git clone http://github.com/markvoelker/blog.git`

The next step is to create a new markdown file in /content/posts.
To keep them more easily organized, I'm prepending the name of
each post file with the date on which it was made.  So for example
I might have one called
[2015-09-25-defcore-misconceptions-1.md](./content/posts/2015-09-25-defcore-misconceptions-1.md).

At the top of each file, you'll want to add a snippet of metadata in
[TOML](http://npf.io/2014/08/intro-to-toml/) format.  That metadata
should include a title, author, date, the relative URL at which you
want it to appear on the site, and a list of categories.  It can
also include other things.  See one of the existing post files for
an example.

The rest of the post file is just the content you want in your post,
written in 
[markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
format.

Once you're done writing, use Hugo to generate the content:

`hugo -t nameoftheme`

Substitute the name of theme you want to use (which can be found,
if you can believe it, in the [themes](./themes) directory).  At
the moment I'm using "redlounge" because I needed to get a post up
when I set all this up and it Just Worked(TM) while not being
obnoxiously busy and generally looking nice and clean.

That done, it's time to save changes and push them up to github
so you don't lose them when you inevitably spill your coffee all
over your laptop (this will eventually happen to every person who is
capable of writing code and endevours to do so).

`git commit -a`

`git push origin master`

Per the workflow I mentioned above, I'm actually keeping the rendered
content in a separate branch which is included in master as a
subtree.  Since the `gh-pages` branch is what GitHub actually renders
into a viewable site, you need to push those changes up too:

`git subtree push --prefix=public git@github.com:markvoelker/blog.git gh-pages`

The [site](http://markvoelker.github.io/blog/) should show your changes
within a few seconds (you may need to shift+refresh to see them).
