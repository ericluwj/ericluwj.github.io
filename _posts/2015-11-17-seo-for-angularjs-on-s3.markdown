---
layout: post
title:  "SEO for AngularJS on S3"
description: "You would not have imagined that merely migrating a AngularJS SPA application onto AWS S3 or any other static file host would have been so troublesome... Today, let us discuss how we enable proper SEO (search engine optimization) for AngularJS on AWS S3."
date:   2015-11-17
---

<p class="intro"><span class="dropcap">Y</span>ou would not have imagined that merely migrating a AngularJS SPA application onto AWS S3 or any other static file host would have been so troublesome. Suddenly, you are faced with issues that cause you tremendous headaches.</p>

*Updated as of 25 November 2015.*

Here are the 2 main issues:

1. Deep linked URLs no longer work if you enable HTML5 mode in your SPA application. In my [previous article]({% post_url 2015-11-10-angularjs-with-html5-mode-on-s3 %}), I provided a quick hack for enabling pretty HTML5 mode URLs in AngularJS on AWS S3 so that deep-linked URLs can now function properly.
2. The quick hack mentioned above is apparently not SEO friendly at all, so it cannot be a long-term solution. Google is actually able to detect the 404 error when AWS S3 is first unable to find the file, and then stops crawling the URL because the 404 error is used to indicate that the URL is no longer in use. This has serious repercussions on your site's SEO. <br>As such, let me explain how we can enable proper SEO (search engine optimization) for AngularJS on AWS S3 as a long-term strategy, even though it will be a lot more effort required.

Actually in this article, I will not just merely address the issue of SEO for Javascript SPA applications on static file hosts, but the more general issue of how to make your Javascript SPA site (placed on a static file host) **crawlable**.

Why **crawlable**? It is an extremely important task to make your site friendly for search engines such as Google, Bing, Baidu, etc. But a lot of times, users would like to share your site or one of your pages on social networks such as Facebook or Twitter, which will crawl your URL to extract some content and obtain a snapshot, but they can't run javascript on your site page. In the case of your AngularJS site URLs, chances are that the social networks are not going to extract much useful content, because most of your content are loaded with Javascript. This does not do much justice to the painstaking efforts you took to develop the site. 

And as of this time of writing, Google is already able to run Javascript while crawling pages, so that is definitely great news, but yet not so for the others such as Facebook and Twitter. *It would hence still be crucial to pre-render or pre-generate some content for all your SPA site pages.*

I know there has been a lot of talk about [Prerender.io](https://prerender.io/) and how it can prerender your web application so that it is SEO-friendly. I would say Prerender.io is actually a very *misleading* service because it claims that it can allow your javascript website to be crawled perfectly by search engines, but that is of course only if you are using a custom server to serve your files. Once you understand how Prerender.io works after much research, you will then discover that it does not actually fit the bill for AngularJS or Javascript SPA applications on static file hosts like AWS S3, Github Pages, and so forth.

Prerender.io provides a dynamic solution to either display the original page or a pre-generated snapshot, but it requires a request to the server for it to work. If you have your AngularJS SPA application on AWS S3 or even Github Pages, you aren't able to relay the request to your server. In other words, Prerender.io is not really a all-encompassing solution for javascript applications, and you definitely can't use it in a straight-forward manner if you are serving your site directly from AWS S3. What you need, is a solution that can dynamically generate snapshots and let them be **accessible on the static site**.

*As a disclaimer, the solution below may not work for certain cases due to different or incompatible coding practices or architectures.*

### Grunt Prerender ###

In a bid for a more automated solution to make my AngularJS SPA application on AWS S3 SEO friendly, I created the [grunt-prerender](https://github.com/ericluwj/grunt-prerender) Grunt tool that creates pre-rendered HTML snapshots for static file serving purposes.

The idea behind the solution is actually very simple. It bears a huge resemblance to the fashion as to which how [Jekyll](http://jekyllrb.com/) generates site pages automatically from blog posts.

Let's say you have a URL `http://www.mysite.com/faq/` that you want to be SEO-friendly. Upon a GET request, a file server will by default look for an index file, e.g. `index.html`, in the `faq` directory of the root directory and serve it, based on the path of the URL. If this is an AngularJS or Javascript SPA application, obviously a `index.html` file will not exist in the `faq` directory, because what should be served by right should be the `index.html` in the root directory.

Hence, a natural solution would be then to place a pre-generated HTML snapshot of the URL `http://www.mysite.com/faq/` as `index.html` in the `faq` directory. The main difference between this pre-generated HTML snaphot at the URL `http://www.mysite.com/faq/` and `index.html` in the root directory will be that the former already has the **content** of the URL `http://www.mysite.com/faq/` populated inside already, unlike the original plain `index.html`.

This solves the SEO for content marketing issue. If the search engine is only able to find the original `index.html` SPA template, whatever content is in it is probably more or less all of the content that will be crawled for your website, so that is almost no SEO.

Using the [grunt-prerender](https://github.com/ericluwj/grunt-prerender) tool, it is now possible to generate all these static HTML snapshots for all relevant URLs that you want to be SEOed, and then to upload them to AWS S3 or any other static file host. For a URL `http://www.mysite.com/faq/`, the tool will take a HTML snapshot of the hashed version of the URL `http://www.mysite.com/#/faq/` and then save it to disk in the right path.

You could simply install [grunt-prerender](https://github.com/ericluwj/grunt-prerender) in your project with:

    npm install grunt-prerender --save-dev

Here are snippets of my own production Grunt workflow to automate this prerendering. What I do is to simply use the [grunt-prerender](https://github.com/ericluwj/grunt-prerender) tool to generate all my HTML snaphots before uploading them back to my production S3 bucket using the [grunt-aws-s3](https://github.com/MathieuLoutre/grunt-aws-s3) tool. If you are already thinking of SEO for your website, I am sure you should definitely have generated a `sitemap.xml` on your site. But in any case, you are still able to input a list of URLs to the tool instead.

{% highlight javascript %}
grunt.initConfig({

  // Empties folders to start fresh
  clean: {
    server: '.tmp'
  },

  aws_s3: {
    options: {
      accessKeyId: localConfig.AWS_ACCESS_KEY_ID,
      secretAccessKey: localConfig.AWS_SECRET_ACCESS_KEY,
      region: localConfig.AWS_S3_REGION,
      uploadConcurrency: 5, // 5 simultaneous uploads
      downloadConcurrency: 5 // 5 simultaneous downloads
    },
    upload_prerender: {
      options: {
        bucket: 'www.mysite.com',
        progress: 'progressBar'
      },
      files: [
        {expand: true, cwd: '.tmp/snapshots/', src: ['**'], dest: '/', params: {CacheControl: 'no-cache'}}
      ]
    }
  },

  prerender: {
    production: {
      options: {
        sitemap: 'http://www.mysite.com/sitemap.xml',
        dest: '.tmp/snapshots/',
        hashPrefix: '!' // it is common for AngularJS applications to enable the hash prefix !
      }
    }
  }
});

grunt.registerTask('s3prerender', function () {
  grunt.task.run([
    'clean:server',
    'prerender',
    'replace',
    'dom_munger',
    'aws_s3:upload_prerender'
  ]);
});
{% endhighlight %}

As a caveat, there are a few pointers you might want to take note to be able to use this solution:

1. It is a good practice to use trailing slashes for URLs (`http://www.mysite.com/faq/` is good, `http://www.mysite.com/faq` is not so good).
2. Be careful when using (Jquery) plugins that do not integrate with AngularJS nicely or update in a two-way binding manner, as they may not function nicely in the prerendered HTML snapshot.
3. All URLs in your application should preferrably be absolute URLs, but if you do have relative URLs, try adding a `<base href="/">` tag.
4. You should probably do some cleaning of your generated HTML snapshots using tools such as [grunt-replace](https://github.com/outaTiME/grunt-replace) or [grunt-dom-munger](https://github.com/cgross/grunt-dom-munger) to finish with nice clean HTML templates. Things you may want to remove include dynamically generated script tags and css styles, as well as AngularJS directives in dynamically generated template content as templates will get reloaded again causing directives to load more than once.

In any case, you may also create your own staging Grunt workflow incorporating the use of this [grunt-prerender](https://github.com/ericluwj/grunt-prerender) tool.

If you already have static pages that you want to be **crawlable**, you may even include this tool in your pre-production workflow.

Last but not least, this tool is still currently a Grunt tool, and is not optimized for its usage. A better use would be as part of a cron task to automate this prerendering.