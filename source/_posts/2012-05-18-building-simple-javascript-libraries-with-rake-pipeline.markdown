---
layout: post
title: "Building simple JavaScript libraries with Rake-Pipeline"
date: 2012-05-18 13:07
comments: true
categories: 
published: false
---
*Cross-posted from [blog.nulayer.com](http://blog.nulayer.com/post/23294069876/building-simple-javascript-libraries-with-rake-pipeline)*

At Nulayer we try to write very modular JavaScript, which allows us to be flexible and re-use those libraries in other projects. But we’ve had difficulty structuring a build process for these small- to medium-sized JavaScript library projects, while meeting the following objectives:

 1. Break it up: While it’s common to write smaller libraries as one big file, we like the mental separation provided by dividing the source into multiple files.
 2. Inline dependency management: We’d rather define each file’s dependencies right in the code than define the concatenation order in some sort of makefile.
 3. Keep files separate while debugging: When we’re setting breakpoints and stepping through code in the debugger we want to see our individual source files, not have to wade through one big file.
 4. A transparent solution: We need the production library to be free from dependency management code.

A year ago, I tried to satisfy our objectives using @jrburke’s require.js. It gave me the multiple source files with dependency management and excellent seamless development (with separate files) and production (combined and minified) environments that I was looking for. Unfortunately at the time, and things may have changed in a year, compiling requirejs out for the production build was not easy. I could get it down to a bare-bones set of require and define functions but it was still there. Requirejs is excellent for large JS projects and I ended up using it extensively for other work but it wasn’t quite the right fit for making a smaller library.

Via the Ember.js project I was introduced to livingsocal’s [rake-pipeline](https://github.com/livingsocial/rake-pipeline) with @wycats’ [rake-pipeline-web-filters](https://github.com/wycats/rake-pipeline-web-filters) and [minispade](https://github.com/wycats/minispade). With this combination I can prefix my source files with minispade “require” calls and satisfy objectives #1 and #2. The minispade filter for rake-pipeline can optionally encode and your source as strings, add a @sourceURL annotation and eval at run-time. In Webkit and Firefox the script debugger will show each eval’d block as a separate file, satisfying objective #3. But minispade is still required.

@wagenet’s addition of the neuter filter goes a long way to satisfying all four requirements. Neuter will determine the correct ordering of all source files and concatenate them into a single file. This is great for a production build as it means minispade is not required. So by using minispade in development and neuter in production, I can meet all four of our project organization objectives… almost.

One of the side-effects of using minispade is that all modules are evaluated within their own closure. This is great if everything that the module needs to share with other modules is exposed through a common global namespace (as in Ember.js) but it’s problematic if I want to define library-wide functions and variables that I don’t want to expose publicly. In my case, I had some helper functions that I put in a common module that was required by a number of other modules in my library. In the neutered production output everything is fine since all modules are appended as-is and can be wrapped in one big shared closure. But in development I was out of luck.

My solution was to use a modified version of the neuter filter that either outputs the raw code (in production) or outputs eval’d strings of the code with @sourceURL annotations (in development). This simple change allows me to satisfy all four of my requirements with very little work.

Here is a snippet of a rake-pipeline Assetfile that can be switched between production vs. development followed by the modified neuter filter (my changes mostly in output_source):

{% gist 2712617 %}

Using rake-pipeline with a modified neuter filter makes it easy to structure, build, and debug JS projects that use multiple source files with dependencies specified right in the code.