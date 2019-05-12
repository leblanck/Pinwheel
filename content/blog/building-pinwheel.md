---
title: "Building Pinwheel with Hugo and Firebase"
date: 2019-05-08T15:04:05-04:00
draft: false
tags: ["hugo", "firebase", "webdev"]
description: "Building Pinwheel with Hugo and Firebase"
meta_img: ""
author: "Kyle LeBlanc"
---

![Hugo & Firebase](/img/hugo-firebase.png)

The website you are reading right now was brought to you with the help of [Hugo](https://gohugo.io/) and [Firebase](https://firebase.google.com/). This combination of tools has allowed me to spin up a live website faster than I ever have before; < 1 hour to a live skeleton site. The free tier of Firebase made it really quick and easy to just pick up and get going, and since it is made by Google, I didn't even need to create a second account.

Hugo makes it *insanely* easy to do dev work on this site by providing some great build tools, live preview, smooth error handling, etc. I really can't recommend them enough for doing some fast static web development. No, I wasn't paid to say this, although I wish I was, but I just really enjoyed using their platform. Okay, enough of that talk - let's get into it!

**Install & Set Up**

Installing Hugo is a breeze - `brew install hugo` (If you don't already use Homebrew or know how much I love it myself, see my [previous post](/blog/automating-new-web-dev-environments/))

Once installed you can spin up a new project/site `hugo new site pinwheel`. This gives you a pretty basic barebones site to work with. The real magic (I think) are in the themes, they make adding CSS to your site so simple it hurts. You can either install it using git from within your site dir or just clone/download the repo and extract/move the theme files into your site yourself.

Pinwheel is currently using a modified version of the [Cocoa-Enhanced](https://github.com/mtn/cocoa-eh-hugo-theme) theme. It has proven itself to be blazingly fast even on slow internet connections:

> "Everything is made here to make the theme really fast to load: inline CSS, deferred Javascript, fonts loaded in an asynchronous way with replacement fonts when they're not loaded, etc. ... Even with a GPRS connection, your blog is readable. With gzip enabled, this theme takes less than 400ms to load entirely, and the content is readable at only 50ms. Also scores 99/100 on Pagespeed."

Your `config.toml` file really becomes the heart of your site. This is where you actually direct Hugo to which theme you are using as well as set global configs for your site (author, desciption, etc)

```toml
baseURL = "https://example.org/"
languageCode = "en-us"
title = "My New Hugo Site"
theme = "ananke"
```

**Page Creation**

Creating new pages in Hugo is pretty intuitive: `hugo new blog/blog-title.md`. This will spin up a markdown page in the blog directory which you can open and edit with your favorite text editor: `cd blog && atom blog-title.md` - I had to brush up on my markdown skills ([this helped](https://guides.github.com/features/mastering-markdown/)) but once I shook the dust off, creating new posts was really fast.




**Build & Publish**

One of my favorite things about Hugo, being that it creates static websites, is that you can spin up a local instance to serve your site using `hugo server`. (Running just `hugo` publishes your site to your specified `publishdir` in your `config.toml` file). This loads your site at `localhost:1313` and any changes made to your site are auto-refreshed at that location.

```bash
Watching for changes in /Users/username/pinwheel/{content,data,layouts,static,themes}
Watching for config changes in /Users/username/pinwheel/config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```
![Hugo & Firebase](/img/localhost.png)

Once all of the dev work is done and I'm happy with where Pinwheel is at, I can compile/build the Hugo site and deploy up to Firebase with nice one-liner from the root of my hugo dir: `hugo && firebase deploy` which will provide the following output when successful -

```bash
| EN  
+------------------+----+
Pages            | 14  
Paginator pages  |  0  
Non-page files   | 15  
Static files     | 27  
Processed images |  0  
Aliases          |  0  
Sitemaps         |  1  
Cleaned          |  0  

Total in 47 ms

=== Deploying to 'pinwheel-7f284'...

i  deploying hosting
i  hosting[pinwheel-7f284]: beginning deploy...
i  hosting[pinwheel-7f284]: found 42 files in public
✔  hosting[pinwheel-7f284]: file upload complete
i  hosting[pinwheel-7f284]: finalizing version...
✔  hosting[pinwheel-7f284]: version finalized
i  hosting[pinwheel-7f284]: releasing new version...
✔  hosting[pinwheel-7f284]: release complete

✔  Deploy complete!

Project Console: https://console.firebase.google.com/project/pinwheel-7f284/overview
Hosting URL: https://pinwheel-7f284.firebaseapp.com
```
**Firebase Console**

The Firebase console has some pretty handy GUI features, all of which I'm sure are available via cl but I haven't done that much digging. Firebase can do a lot, but I'm only using the Hosting portion and the dashboard for that is simple but convenient. It shows a helpful deploy history as well as basic domain settings. They make adding a custom domain front and center which was a blessing to not need to dig in menus for that.

![Firebase Console](/img/firebase-console.png)

*Resources*

* [Hugo Docs](https://gohugo.io/hosting-and-deployment/hosting-on-firebase/)
* [Firebase Docs](https://firebase.google.com/docs)
* [Theme Repo](https://github.com/mtn/cocoa-eh-hugo-theme)
