---
layout: post
title:  "AngularJS with HTML5 Mode on S3"
description: "How do you fix AngularJS SPA application with HTML5 mode enabled on Amazon S3? More effort is required for pretty URLs."
date:   2015-11-10
---

<p class="intro"><span class="dropcap">F</span>or my latest project [Jobbies](http://www.jobbies.co/), I had it hosted on a simple static file server AWS S3, but it was soon found out that with HTML5 pushstate enabled, some links could not work properly once the site was up on AWS S3.</p>

Just for an example, the link `http://www.jobbies.co/faq/` no longer works by default, because the S3 file server will try to serve `http://www.jobbies.co/faq/index.html` or any other possible index files in the `faq` directory of the S3 bucket where the site was hosted on. However since the `faq` directory does not exist in the first place, a 404 error is returned.

### AngularJS HTML5 Mode ###

Usually by default, we enable the HTML5 mode to prettify our URLs (remove the hashbang symbols like `#` or `#!`), but this mode is not compatible with file servers by default. Earlier, we already gave an example to show the problem. If the browser does not support HTML5, it should fall back to using the hashbang prefix (`#!`) for URLs. To do so, we need the following line in the AngularJS app configuration.

{% highlight javascript %}
angular.module(app, [])
  .config(function($locationProvider) {
    $locationProvider.html5Mode(true).hashPrefix('!');  
  });
{% endhighlight %}

### AWS S3 ###

Now that you have set the AngularJS configuration right, you need to set the AWS S3 configuration right too.

The following conguration data should be inserted into the Redirection Rules under 'Static Website Hosting' for the S3 bucket serving your site. Do replace `%HOST_NAME%` with your website host name.

{% highlight xml %}
<RoutingRules>
  <RoutingRule>
    <Condition>
      <HttpErrorCodeReturnedEquals>404</HttpErrorCodeReturnedEquals>
    </Condition>
    <Redirect>
      <HostName>%HOST_NAME%</HostName>
      <ReplaceKeyPrefixWith>#!/</ReplaceKeyPrefixWith>
    </Redirect>
  </RoutingRule>
</RoutingRules>
{% endhighlight %}

You can say that this is still kinda like a hack to fix the pretty AngularJS URLs. What it essentially does is if the S3 file server can't find the exact file and throws a 404 error, the S3 hosting server will catch it and does a redirection with `#!/` appended between the host name and the URL path, which will allow it to serve the main application `index.html` and hence avoid any issues.

Even though this hack has managed to fix pretty AngularJS URLs, which is already pretty much an achievement in itself, it is **not a SEO-friendly solution**. But fortunately, I have created another solution to complement this workaround for allowing search bots to search your site properly. You can now find this solution [at this page here]({% post_url 2015-11-17-seo-for-angularjs-on-s3 %}).