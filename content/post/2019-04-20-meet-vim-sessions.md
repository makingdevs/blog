+++
tags = ["vim", "developer tools", "sessions"]
draft = false
date = "2019-04-20"
title = "Meet Vim Sessions"
description = "Meet Vim Sessions one handy tool for switching between projects and branches without loosing your current work"
author="sohjiro"
+++

A few weeks ago, while I was tracking and debugging a system in which I'm currently working, the client told me that another part of the system was having troubles.

Suddenly, I'm on verge of close all the **Vim**'s buffer I've just opened because I need to switch branches meaning that I need a way to know in which files I was working before closing them.

Needless to say, it was hard to found those files and I even haven't made a change in them (I have just found them) Additionally, is a huge project with a lot of micro projects so, at this point, I didn't know in which project I was working (I know my mistake).

With this in mind, I have a couple of options to keep them. However, many of them implies to write the name of the files or services and to be honest I don't want to maintain another file, at least not in that way.

In that moment, I remembered an _unknown_ feature in **Vim** or at least the less used, which I barely know one or three people that already knew about it.

That feature is _**Sessions**_

---

### Sessions

Before going further, you need to know that a `session` is a collection of something called `views` and at the same time a `view` is a collection of something, in this case settings that apply to one window. Believe or not, you can save a `View` (collection of settings) and when you restore it later, the text is displayed in the same way. Options, mappings and everything else will also be restored. That means that you can continue editing like when the `View` was saved.

If you read carefully, you might figure out the behavior of a `session`; if not, don't be afraid and let me explain it you.

And a `session` is the same but for `views` i.e. a `session` keeps the `Views` for all windows with its settings and the global ones. When you save a `session` and restore it later, the window layout will looks the same. You can use this for quickly switch between different projects (or in my case, branches). With this, you can automatically load the files you were last working in that project (or branch).

But, How do I use it? Well, that's one of the easies thing to do in Vim (weird, isn't it?) Just need to run the next command and that's all.

![][1]

I must say that ones enclosed with braces (`[]`) are optionals including filename which by default is `Session.vim`

Let's take the following three files as an [example](https://gist.github.com/sohjiro/591f7ffa400e0eb7efa584fda68bf936) and create the following layout (if you have troubles creating this layout, you should read the following posts [Introduction to Vim buffers](https://medium.com/@Sohjiro/introduction-to-vim-buffers-dd966ff518d) and [Buffers and Windows in Vim](https://medium.com/@Sohjiro/buffers-and-windows-in-vim-c7fecfbc473c):

![][2]

Now let's save this `session` with the command that we saw before and close `Vim`

![][3]

Well that's awesome but, how do I load this `session`? Well, once you have created you should load when `Vim` starts with the following command.

![][4]

And that's all! Now you have the same files and layout just as before quitting `Vim`.

## Conclusions

Through this post, we have learned a new feature that has shown to be pretty handy to deal with switching projects (or branches) Also we recap a little about buffers. Even though it’s a short post, I hope it can help you on your way to reveal the amazing features a tool like Vim has.

But for this post we are done. I hope you find it interesting and helpful. and as usual, If you have any comments or any doubts please let me know.

See you next time. Good luck and have fun!



[1]: /sessions/make_session_structure.png
[2]: /sessions/layout_example.png
[3]: /sessions/mks.png
[4]: /sessions/load_session.png
