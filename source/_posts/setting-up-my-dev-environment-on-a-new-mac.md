---
title: Setting up my dev environment on a new Mac
tags:
  - alfred
  - git
  - homebrew
  - iterm
  - osx
  - terminal
  - tools
id: 15
categories:
  - dev
date: 2015-02-12 10:00:27
---

Here's what I do when I get a new Mac / reinstall OSX.

### The absolute essentials

1.  Download [chrome](https://www.google.com/chrome/). Already, everything is synced. Awesome.
2.  Download [Alfred](http://www.alfredapp.com/). This is my go-to tool for opening just about anything. It's so good I [actually bought](http://meltuhamy.com/tech/dev/why-buy-alfred-powerpack/ "Why I bought the Alfred Powerpack") the powerpack.
3.  Download [Spectacle](http://spectacleapp.com/) for easy and powerful window management.
4.  Download [iTerm](http://iterm2.com/index.html). This is my preferred terminal environment.
5.  Set up a [global keyboard shortcut](http://computers.tutsplus.com/tutorials/how-to-launch-any-app-with-a-keyboard-shortcut--mac-31463) for opening iTerm.
6.  Install [homebrew](http://brew.sh/). Once we have this, we have everything. When installing homebrew, it will also install the Apple Command Line Developer Tools. Yay.
7.  brew install all the things. I normally brew install node first as I am a self-proclaimed JavaScript fanboy.
8.  Install [Sublime Text 3](http://www.sublimetext.com/3). Unless you're a vim wizard. In that case ignore steps 1-7\. Vim will [suffice](https://github.com/ryanss/vim-hackernews).

### Make the terminal awesome

![Dat terminal do](https://raw.githubusercontent.com/sindresorhus/pure/master/screenshot.png)

1.  Download [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh). I can't be bothered explaining the [benefits of zsh over bash](http://www.slideshare.net/jaguardesignstudio/why-zsh-is-cooler-than-your-shell-16194692) but you'll feel the power of zsh as soon as you start using it.
2.  Set up [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting). Gives instant visual feedback to tell you what you're about to execute is correct. e.g. If you type the command "echo" incorrectly, it will show in red. If you type it correctly, it will show in green (like the above screenshot), _before_ you actually execute the command.
3.  Download [tomorrow-night-eighties](https://github.com/chriskempson/tomorrow-theme/tree/master/iTerm2) theme for iTerm2\. To do this, save [this file](https://raw.githubusercontent.com/chriskempson/tomorrow-theme/master/iTerm2/Tomorrow%20Night%20Eighties.itermcolors) as 'Tomorrow Night Eighties.itermcolors' and open it. iTerm2 will import it. Then, choose it in iTerm &gt; Preferences &gt; Profiles &gt; Default &gt; Colors &gt; Load Presets...
4.  Set up [pure prompt](https://github.com/sindresorhus/pure).  This will add stuff like nicer git integration, timing functions (see screenshot above), and other neat tricks in your terminal. To do this, save [this file](https://raw.githubusercontent.com/sindresorhus/pure/master/pure.zsh) as 'pure.zsh'. Then run:
```
mkdir ~/.oh-my-zsh/functions
ln -s /path/to/pure.zsh ~/.oh-my-zsh/functions/prompt_pure_setup
```
5.  Restart iTerm.
6.  At this point, I like to set up my zshrc aliases. Sublime Text is an important one. Add this to the end of your .zshrc file:
```
alias subl="'/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl'"
```
Cool, you can now edit files in your terminal using the subl keyword! (Try it on folders too!)

### Other stuff

Set up your ssh keys:

1.  Open a terminal and type ssh-keygen
2.  Repeatedly press enter (feel free to give a password if you want)
3.  Copy your public key and put it into your [github account](https://github.com/settings/ssh).
`cat ~/.ssh/id_rsa.pub | pbcopy`

Set your git user name and email:

```
git config --global user.name "Your Name"
git config --global user.email you@example.com
```

Set up git lg alias, [a better git log](https://coderwall.com/p/euwpig/a-better-git-log):

```
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)&lt;%an&gt;%Creset' --abbrev-commit"
```

### [JIT](http://en.wikipedia.org/wiki/Just-in-time_compilation) App Installations.

A lot of people install all the apps they could possibly need in the future right after they do a fresh install. I used to do this. Then I realised the majority of this time is time wasted. A better approach is to install apps on the fly when you need them. This will save you time, and also some precious disk space! Package and app managers (like npm or the App Store) make installations really quick and easy. So install the essentials and forget the rest!