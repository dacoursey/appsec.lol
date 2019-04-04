---
layout: post
title: "Removing Bad Cached Git Credentials"
author:
modified:
tags: [Development,NoteToSelf]
---

I'm probably going to forget this in three minutes so I'm writing it down.

I needed to log into github from a different account to help somebody out. After I was done in VS Code I went back to my own work and I couldn't push new code. The error message (Cmd+Shift+U) 
showed me that the temporary user account was now somehow the account that was being used for my personal project. I checked `~/.gitconfig` and it was the proper creds. I tried forcing some change
with GitHub Desktop, and that didn't help. I could still use GH Desktop because it stores creds separately, but I don't want to have to open that every time just to push.

I finally found a page on GitHub Help that said the keychain can be used as well:  

[https://help.github.com/articles/updating-credentials-from-the-osx-keychain/](https://help.github.com/articles/updating-credentials-from-the-osx-keychain/)


I also discovered that `git-credential-osxkeychain` will use the first account listed, which happened to be the last one that was modified.

You can remove the invalid entry by opening Keychain Access or directly from the CLI:

```shell
$ git credential-osxkeychain erase
host=github.com
protocol=https
```