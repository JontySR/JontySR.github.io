---
layout: default
permalink: "/faq/"
---

# Frequently Asked Questions

## How do I ssh into the CS server?

Type <code>ssh tinky-winky.cs.bham.ac.uk</code> and input your username and password.

## Why can't I use gedit over ssh?

You can (put -X after ssh in the command above), but you shouldn't (see below). The reason you can't without -X is that gedit is a graphical application, and the only thing being directed through ssh is the terminal screen you see.

## Why shouldn't I direct the display server through ssh?

Because it's sending all of that window information over the internet, and that's gonna be laggy and unusable.

## What can I use for editing my files on the CS server then?

- scp (copy the files over via ssh) ([See this page for more info](https://www.linux.com/learn/intro-to-linux/2017/2/how-securely-transfer-files-between-servers-scp))
- sshfs (setup a virtual ssh filesystem so you can access your stuff with your native applications) ([See this page for more info](https://github.com/libfuse/sshfs))
- Use git, it's really good at doing this stuff and you'll need to learn it at some point. Create an account at gitlab.com or github.com and set up a new repository. ([See this page for git commands and some brilliant explanations](http://rogerdudler.github.io/git-guide/))

## This right here is crazy talk. Why can't I just use a USB like everyone else?

- You'll have to remember to transfer everything over every time you work.
- Everyone loses their USBs by leaving them plugged in.
- You're a CS student and deserve more efficient ways to improve your workflow.
- Bragging rights.
- You'll probably have to use one of the above at some point and it will be under practical circumstances, so do it now.

## Git is denying access to the repo for the first time

Log into [the GitLab login page](https://git.cs.bham.ac.uk) first.

## I did, and it's still denying me access

Type `git config --global --unset credential.helper` and
`git config --system --unset credential.helper` (to make
sure). This is because git has cached your credentials for a while.

## Is [insert session here] on today?

Check canvas announcements, canvas groups, your timetable, etc.