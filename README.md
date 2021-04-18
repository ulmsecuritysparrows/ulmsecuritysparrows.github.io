# Ulm Security Sparrows Blog Post Workflow

### Knowledge prerequisites for participation in the blog post authoring.

* Simple cmd line handling
* [Git VCS](https://git-scm.com/book/en/v2), if not it's about time
* [Markdown](https://docs.gitlab.com/ee/user/markdown.html)
* [Gihub (Pull Requests)](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)

### TL;DR

Work in the devel branch only. Look how other posts are created. Keep that
format or be prepared to listen to some ranting.
The only dependencies are Ruby and Bundler. Use ``bundle install`` after
cloning the repo. Afterwards run ``bundle update`` before starting to write a
blog post. Run ``bundle exec jekyll serve`` to build and preview your changes.
Commit, push, collaborate and let others read over your text. When the blog post
is done, create a merge request in Gitlab to the master branch and let it be merged by one of the
project owners. She will ensure you stuck to the workflow, rename your post
and its assets to the merging date and, of course, merge your request into the
master branch, if it is okay.

If you are doing this for the first time clone the project and change to devel
branch.

```bash
git clone git@github.com:ulmsecuritysparrows/blog
cd blog
git checkout devel
bundle exec bundle install
```
If you want to contribute to the blog again, you can do the following.

```bash
cd path/to/blog
git pull
bundle update
```
Create your blog post, add, commit and push it. Git workflow as usual.

```bash
git add _posts/2017-01-27-somectf-writeups.md
git add assets/2017-01-27-somectf-writeups/poc_xy.png
git commit -a -m "Initial somectf writeup. Added xy-Challenge"
git push
```

If your contribution to the blog post is ready, create a pull request to the devel-branch.
When a repo owner/master/employee accepts your request and merges your changes
into the master branch they automatically become available on the [USS blog](https://uss.informatik.uni-ulm.de).


### Working with Jekyll

Jekyll is a framework for static content generation. One writes blog posts in
Markdown, which are converted to HTML and published. To achieve this you need to
install Ruby and Bundler. The latter gives you the opportunity to install
dependencies, like Jekyll, locally. Avoid installing Jekyll globally. Since every
OS offers different versions (some outdated) of the framework this helps us
to build and preview the blog posts on different machines, including the web
server.

Install Ruby and Bundler. The latter is a Gem (Ruby speak for library). You'll
find the packet pretty much on every Distro under the name bundler, ruby-bundler,
rubygem-bundler or alike. If you're using OSX or Windows then you're probably
looking for the command ``gem install bundler``.

After cloning the blog repository run ``bundle install``. This will
install Jekyll and all its dependencies locally (usually into the folder vendor).
This is needed only the first time you're creating a post/cloning the repo.
If you write blog posts any time again make sure you run ``bundle update`` beforehand.
This will update dependencies of Jekyll (if any new exist) or even Jekyll itself.
The latter happens if the version in the *Gemfile* is changed. This is done
regularly by the repository owners. That is why it is important to bundle
update!

> The Gemfile, as also Jekyll config files are ignored during the build process on the web server.
> This applies to the devel, as also to the master branch.
> If you feel the need to bump the admins you can update them and make a merge request
> Still, the admin of the server must update them by hand. Only then the changes will be applied on the web server.


Jekyll holds all the blog posts inside the folder *_posts* as Markdown files.
If you run ``bundle exec jekyll build`` then the resulting HTML output is
generated inside the *_site*. You'll probably end up using the command
``bundle exec jekyll serve`` since this not only builds the blog but also starts a
build-in web server for preview. If you edit files while serve is running,
rebuilds are triggered automatically.

Sidenote: ``bundle exec`` enforces the usage of locally
installed gems.

### Create a New Post

- Create a .markdown file inside *_posts* folder.
- Name the file according to the standard jekyll format.
```
   2016-03-30-i-love-design.markdown
```
- Write the Front Matter and content in the file.
    ```
          ---
          layout: post | default | page
          title:  String Post Title
          date:   Time Stamp
          categories: String | Array of Strings Category / Categories
          ---
    ```

    ```
        ---
        layout: post
        title:  "The One with the Blackout"
        date:   2016-03-30 19:45:31 +0530
        categories: ["life", "friends"]
        ---
    ```
- If you want to add assets (pictures or attachments) to your blog post then create the according folder in ``assets``.

```
  assets/2016-03-30-i-love-design/design-example.png
```

#### Choosing a Date

If you're writing a large document, especially in collaboration with others,
you can't guess the day it is going to be published - pick current date anyway.
The date is adjusted during merge if necessary.

#### Contribution for Write Ups

People writing CTF write ups spend a lot of time for PoCs, structuring it into
text and hopefully QS. Feel free to add your name underneath the post, link to
your website or a socnet account. Also note which sections are written
by yourself. BUT please keep it minimal: a name with **one** link and authored section.

Example:

This write up is brought to you by,

* [Hans Mueller](https://uss.informatik.uni-ulm.de/): Section ChallengenameXXX, ChallengenameYYY
* [Joe Doe](https://uss.informatik.uni-ulm.de/): Section ChallengenameZZZ

