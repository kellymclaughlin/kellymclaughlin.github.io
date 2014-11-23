---
layout: post
title: Setting up the Emacs daemon on OS X
---

*Warning:* I only use terminal emacs so I have no idea how well this will work otherwise.

## Install emacs 23 or later.
Here is what I use to get the very latest emacs goodies:

```
sudo brew install emacs --cocoa --use-git-head --HEAD
```

Some people have had trouble with the `--cocoa` flag so you may find
it convenient to omit that if you do not need the Cocoa version. Also
if you do not want the bleeding edge version, then omit the
`--use-git-head --HEAD` options.

## Create a plist file and move it to ~/Library/LaunchAgents
Here is what my file looks like:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
  <plist version="1.0">
    <dict>
      <key>Label</key>
      <string>gnu.emacs.daemon</string>
      <key>ProgramArguments</key>
      <array>
        <string>/usr/local/Cellar/emacs/HEAD/Emacs.app/Contents/MacOS/Emacs</string>
        <string>--daemon</string>
      </array>
      <key>RunAtLoad</key>
      <true/>
      <key>ServiceDescription</key>
      <string>Gnu Emacs Daemon</string>
      <key>UserName</key>
      <string>kelly</string>
    </dict>
  </plist>
{% endhighlight %}

Update the path the emacs and change the username to your OS X user name.

## Start the daemon
```
launchctl load gnu.emacs.daemon
launchctl start gnu.emacs.daemon
```

### Edit files with emacsclient instead of emacs

Instead of opening a file with `emacs <filename>` instead use `emacsclient -nw <filename>`.

I created a handy alias in my `.bashrc` file.

```
alias e="emacsclient -nw"
```

### Stopping the daemon

From within emacs, use the `save-buffers-kill-emacs` command.

Outside of emacs:

```
emacsclient -e '(save-buffers-kill-emacs)'
```

### Why bother?

Setting up the daemon has a several advantages.

1. Files load much faster.
1. You can share buffers among different `emacsclient` instances.
1. If you exit all the `emacsclient` instances and come back later,
your buffers are all still open. When you access a buffer the cursor
will be at the exact spot you left off previously.

### Downsides

The server does occasionally crash, but that is probably because I am
using the bleeding edge version. It hasn't caused me to lose anything
thanks to auto-saving and file recovery so go for it.

### References

* [EmacsWiki](http://www.emacswiki.org/emacs/EmacsAsDaemon)
