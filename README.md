# Vagrant

This is a Vagrant configuration plus provisioning shell script that lets you easily `vagrant up` a local Jekyll-powered GitHub pages server. It accompanies blog post: [Vagrant, Jekyll and Github Pages for streamlined content creation](http://kappataumu.com/articles/vagrant-jekyll-github-pages-streamlined-content-creation.html).

Getting started is straightforward:

```
host$ sed -i 's#XXX#https://github.com/kappataumu/kappataumu.github.com.git#' bootstrap.sh
host$ vagrant up
host$ vagrant ssh
guest$ cd /srv/www/
guest$ npm install --no-bin-links
guest$ ./node_modules/grunt-cli/bin/grunt watch
guest$ jekyll serve --watch --force_polling
```

# Clean Blog by Start Bootstrap - Jekyll Version

The official Jekyll version of the Clean Blog theme by [Start Bootstrap](http://startbootstrap.com/).

## Before You Begin

In the _config.yml file, the base URL is set to /startbootstrap-clean-blog-jekyll which is this themes gh-pages preview. It's recommended that you remove the base URL before working with this theme locally!

It should look like this:
`baseurl: ""`

## What's Included

A full Jekyll environment is included with this theme. If you have Jekyll installed, simply run `jekyll serve` in your command line and preview the build in your browser. You can use `jekyll serve --watch --force_polling` to watch for changes in the source files as well.

A Grunt environment is also included. There are a number of tasks it performs like minification of the JavaScript, compiling of the LESS files, adding banners to keep the Apache 2.0 license intact, and watching for changes. Run the grunt default task by entering `grunt` into your command line which will build the files. You can use `grunt watch` if you are working on the JavaScript or the LESS.

You can run `jekyll serve --watch --force_polling` and `grunt watch` at the same time to watch for changes and then build them all at once.

## Support

Visit Clean Blog's template overview page on Start Bootstrap at http://startbootstrap.com/template-overviews/clean-blog/ and leave a comment, email feedback@startbootstrap.com, or open an issue here on GitHub for support.

# Jekyll Themes

Good [Jekyll Themes](http://jekyllthemes.org/)

10  http://ironsummitmedia.github.io/startbootstrap-clean-blog-jekyll/
9   http://niklasbuschmann.github.io/contrast/
9   http://pixyll.com/
9   http://tybenz.com/
8   http://yulijia.net/freshman21/
7   http://www.jacoporabolini.com/emerald/
7   http://brianmaierjr.com/long-haul/index.html
7   http://aigarsdz.github.io/brume/
7   http://chibicode.github.io/solo/
7   http://richbray.me/frap/gt/
6   http://phlow.github.io/feeling-responsive/
6   http://hmfaysal.github.io/hmfaysal-omega-theme/
6   http://mmistakes.github.io/hpstr-jekyll-theme/
6   http://jekyllthemes.org/themes/solar/
5   http://madebygraham.com/midnight/
4   http://vinitkumar.me/white-paper/
