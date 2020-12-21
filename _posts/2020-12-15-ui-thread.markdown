---
layout: post
title:  "Android Gallery scrolling behind the scene!"
date:   2020-12-21 15:05:01
categories: android
---
# Android Gallery scrolling behind the scene!

If you ever scrolled photos on the screen of an Android device did you think of the implementation of it? You see it very smooth to scroll.
Overall it’s a very complicated process it requires heavy converting tasks in the background and putting the result into a view screen.
And moreover, it should be done in the most efficient way.
The only thread that is responsible for drawing anything on screen for a particular application is the main or UI thread.
So let’s find out how it happens.

![Demo]({{ site.url }}/assets/galleryscroll.gif)

## How it works

### Problem
The *main* thread shouldn't be blocked. Converting from .jpg file or any other file to a thumbnail picture is a heavy task.
That's why any converting should be done outside of the main thread. The problem is solved by applying AsyncTask and caching.
And to be more efficient we should use the reused element *View* of the *GridView*.

The application could work with any amount of images and any size of them.

### AsyncTask
*AsyncTask* could be used to start the process outside of the main thread.
It's part of Android SDK. For each new *View* element of *GridView* we check the thumbnail image in the cache,
if there is no such we should start this task.
The task could be canceled if *View* already outside of the screen, and the task wasn't started yet, it could be in case of fast scrolling.
The task converts .jpg to thumbnail format   
**Note.** AsyncTask was deprecated from API level 30.

### Caching
There are two caching ways: storing thumbnails in the memory and storing them on the cache directory.
First, we check in memory, and only then after we check in the cache directory.
The mechanism for the cache is LRU(least recently used), i.e. we try to store in cache most usable thumbnails.

### Reused View
For the most complex *View*s like *GridView* adapter provides you possibility to reuse *View* element in
```
    public View getView(int position, View convertView, ViewGroup parent)
```
If you check the documentation it's saying:
> convertView - The old view to reuse, if possible. Note: You should check that this view is non-null and of an appropriate type before using. If it is not possible to convert this view to display the correct data, this method can create a new view.

It's done for more efficient way of using elements outside of the screen. It's your obligation to check if you could reuse the same view.
In our example we could reuse it, otherwise we create new one. But for both reusable and new one we should set image for the current position.

### Overall mechanism
To combine it all together we have the next logic.
![Schema of working]({{site.url}}/assets/Schema.png)

We are scrolling up, i.e. new elements will come up from the bottom, old elements will disappear at the top.
And for new *View* we should provide a thumbnail. Adapter receives reused *View*, we are reusing it if exists.
We set new *ImageView*, first, we check the in-memory cache.
Before starting the new *AsyncTask* we should check the previous task associated with this view, and cancel it if it was obsolete.
Then next step is made in the background thread, i.e. outside of the main thread. We check the cache, and if there is no then create a thumbnail and put it to the cache.
And in the main thread, we set *ImageView* if it wasn't changed, because if the task took longer and the element reused again, then we shouldn't put an image in this *View*.

## Conclusion

As we can see the logic is quite complex. Because we try to make it more efficient. And that's why we use
* reuse *View* on *GridView*
* using LRU cache in memory and disk
* using asynchronous task

And as the result, we have a smooth scrolling gallery.

[Here is sources of the application](https://github.com/oleg-sta/GalleryScroll)