---
layout: post
title:  "Configuring an Application Load Balancer on AWS Elastic Beanstalk"
description: ""
date:   2016-12-07
---

<p class="intro"><span class="dropcap">W</span>ith the new application load balancer on the AWS Elastic Beanstalk, it is finally much easier to run an application that incorporates both the client and server code, such as applications based on the MEAN stack.</p>

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- ericluwj.com -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3690983003821502"
     data-ad-slot="1621079697"
     data-ad-format="auto"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

AWS Elastic Beanstalk used to only support a very basic **classic load balancer** which does pretty much a good job. But with respect to new full stack applications that incorporate both an API server and a web server that renders the client side application, the classic load balancer is clearly lacking in functionality.

So here some of the powerful features that the **application load balancer** has that surpasses the **classic load balancer**:

* It can point different URL paths to different ports on the application instances.
* It support sockets which are really very useful for realtime apps.
* It provides HTTP/2 support which allows for amalgamation of various HTTP requests 

It is important to take note that a lot of AWS Elastic Beanstalk documentation still refers to the **classic load balancer** as the *load balancer* while the **application load balancer** actually has a whole set of different configuration options compared to the classic load balancer. For an example, the entire documentation at [http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.elb.html](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.elb.html) is referring to just the classic load balancer, just in case you mistook the documentation for referring to both types of load balancers in general.

For the worst part, the **Application health check URL** within the "Health" configuration settings did not actually reflect the right setting for the **application load balancer**. I had to set the right setting using configuration files.

<img src="{{ "/assets/img/elb.png" | prepend: site.baseurl }}" />

For most applications, it would be very common to perform 2 sorts of configuration:

#### Modifying health check and stickiness settings

{% highlight yaml %}
option_settings:
  aws:elasticbeanstalk:environment:process:default:
    DeregistrationDelay: '20'
    HealthCheckInterval: '15'
    HealthCheckPath: /
    HealthCheckTimeout: '5'
    HealthyThresholdCount: '3'
    UnhealthyThresholdCount: '5'
    MatcherHTTPCode: null
    Port: '80'
    Protocol: HTTP
    StickinessEnabled: 'true'
    StickinessLBCookieDuration: '43200'
{% endhighlight %}

#### Adding (default) HTTPS to your application

{% highlight yaml %}
option_settings:
  aws:elbv2:listener:443:
    DefaultProcess: https
    ListenerEnabled: 'true'
    Protocol: HTTPS
    SSLCertificateArns: arn:aws:acm:us-east-1:0123456789012:certificate/21324896-0fa4-412b-bf6f-f362d6eb6dd7
  aws:elasticbeanstalk:environment:process:https:
    Port: '443'
    Protocol: HTTPS
{% endhighlight %}

For more information, you may check the full documentation at [http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-cfg-applicationloadbalancer.html](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-cfg-applicationloadbalancer.html).
