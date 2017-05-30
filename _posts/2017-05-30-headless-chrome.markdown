---
layout: post
title: Headless Chrome Basic Auth
categories: [javascript]
---

I recently spent some time checking out [Headless Chrome](https://developers.google.com/web/updates/2017/04/headless-chrome) since it seems it will probably at least kill [PhantomJS](http://phantomjs.org/).

It doesn't yet fully appear ready for prime time but, [there are some cool uses](https://medium.com/@dschnr/using-headless-chrome-as-an-automated-screenshot-tool-4b07dffba79a) already in the wild.

All I wanted to do was hit a page that had basic auth enabled, but it turned out to be more of a task than I thought. Here are some of my notes:

* Get Ubuntu box up and running.

* Install latest Chrome:

(From [askubuntu](https://askubuntu.com/questions/79280/how-to-install-chrome-browser-properly-via-command-line))

```
sudo apt-get install libxss1 libappindicator1 libindicator7
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome*.deb
sudo apt-get install -f
sudo dpkg -i google-chrome*.deb
```

* Test Headless Chrome:

```
google-chrome-stable --headless --disable-gpu --no-sandbox --remote-debugging-port=9222 --remote-debugging-address=10.138.33.145  http://www.google.com
```

Navigating to that port/IP should display a page you can debug your Chrome session on.

* Install node, npm, chrome-remote-interface.

* Set up [chrome-remote-inteface](https://www.npmjs.com/package/chrome-remote-interface) node script.

* Use [Chrome DevTools docs](https://chromedevtools.github.io/devtools-protocol/tot/Network/#type-Headers) to browse options.

* Come up with something like [this repo here](https://github.com/hross/headless-chrome-basic-auth).
