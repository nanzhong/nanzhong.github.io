---
layout: post
title:  Better Email with mu4e
date:   2016-10-03 21:51:54 -0400
tags:   [emacs, email, mu4e]
---
I've never been a big fan of email, but at this point, it's so ingrained into the day to day of every profession that the only thing one can do is conform. I have been using gmail ever since it was introduced my Google in 2004, and I have to admit that out of all the free email providers, it really does a great job of making email more enjoyable. Sure its UI has had its ups and downs, and it has been the centre of some controversy with the scanning of emails for advertisements, but the feature set that it provides is really very complete and work quite well when you are dealing with reasonably low volumes of email.

As the amount of email you receive starts to increase, things like:

- labels, 
- custom filtering rules, 
- complex search queries, 
- and batch actions

start to play an important role in the way you deal with email, and gmail handles this quite well. However, I found that this type of generic email workflow really starts to fall apart as you start receiving more and more email.

Why so much email in the first place? Well, I can only speak for myself, but as a software engineer, my email consists of:

- github notifications from public and enterprise (**~50/day**)
- error notifications (**~30/day**)
- mailing list discussions (**~30/day**)
- actual relevant email (**~10/day**)
- random other things email (**~10/day**)

I'm sure I can reduce the sheer amount of email by spending some time going through things such as unwatching github repositories I'm no longer actively contributing to, but that won't change the total amount very significantly and feels much more like a workaround than actually trying to solve the problem.

Another huge annoyance is how disruptive checking email via the browser or a dedicated email client is. I spend most of my day staring at emacs, and when emails notifications ding, I'm always distracted and want to check; switching to the browser pulls me out of my flow. Browsers are also notorious for being huge distractions tempting you to check [hacker news](https://news.ycombinator.com) or [reddit](https://reddit.com) (at least for me), and we all know how easily time is lost "just checking if anything new popped up on the front-page".

# mu4e

Addressing these annoyances is where [mu4e](https://www.djcbsoftware.nl/code/mu/mu4e.html) comes into play. mu4e is an email frontend for emacs that built on top of mu, which is an email indexer/searcher. My reasons for why mu4e has greatly improved my email experience are:

- super fast and optimized UI
  - keyboard driven actions
  - bulk selection/actions can leverage all of emacs' search/marking functionality
- fast search supporting complex querying
- extensible because emacs :fireworks:
- keeps me in my workflow since I'm already in emacs
- no distractions

*Note: there are a number of other text based email clients which are similar to mu4e (another client I've tried for a couple months was mutt), but I feel like mu4e is much better overall; it so much more customizable and fits really nicely with the way gmail's imap implementation (more on this later).*

Getting mu4e up and running is not a difficult task, but it's definitely more work than you are probably used to if you haven't tried other text based email clients before. The following is description of my configuration and setup that hopefully you should be able follow along with and get yourself up to speed.

My email setup is specifically for 2 gmail accounts, one personal and one for work, both of which are on Google Apps and secured via Two-Factor Auth. The steps below are also tailored to macOS. If your setup is different, much of the following should still be relevant with a few tweaks.

# Syncing Email

mu4e/mu works off of a [maildir](https://en.wikipedia.org/wiki/Maildir), so the first step is to actually get your email accounts setup to sync with your local maildir. It feels a little weird to be storing copies of your email locally in this day and age, but one often overlooked advantage to this is that you always have full access to your historical email (eg. when you don't have internet access). There are a number of applications you can use for this: [offlineimap](http://www.offlineimap.org/), [mbsync](http://isync.sourceforge.net/), and others. I also use my phone to check my email, so being able to sync email state between devices is a must; this rules out using POP and leaves me with IMAP. I personally offlineimap because it seems to be the most widely used, and it seems to be the only one that support XOAUTH2 which is the preferred way to access gmail accounts.

First, install offlineimap

{% highlight sh %}
brew install offlineimap
{% endhighlight %}

The next step is to configure offlineimap. It looks for the configuration at `~/.offlineimaprc` so that where the config will need to live. Before adding the configuration. offlineimap also support configuration that is evaluated using python; any `import`s and any other things such as custom function definitions should all live there. I take advantage of this feature to pull my oauth credentials from the macOS keychain so that they don't need to be stored in plaintext in the config file. My python script at `~/.offlineimap.py` contains:

{% highlight python linenos %}
import json
import subprocess

def secure_string_for(account, service, value):
    return json.loads(subprocess.check_output(["security",
                                               "find-generic-password",
                                               "-a", account,
                                               "-s", service,
                                               "-w"]).strip())[value]
{% endhighlight %}

# Sending Email

# Putting it All Together with mu4e

![mu4e](/assets/mu4e.png)
