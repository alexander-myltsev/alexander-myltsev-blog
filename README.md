This is a Vagrant configuration plus provisioning shell script that lets you easily `vagrant up` a local Jekyll-powered GitHub pages server. It accompanies blog post: [Vagrant, Jekyll and Github Pages for streamlined content creation](http://kappataumu.com/articles/vagrant-jekyll-github-pages-streamlined-content-creation.html).

Getting started is straightforward:

```
$ sed -i 's#XXX#https://github.com/kappataumu/kappataumu.github.com.git#' bootstrap.sh
$ vagrant up
```
