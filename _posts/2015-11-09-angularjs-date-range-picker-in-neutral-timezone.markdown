---
layout: post
title:  "AngularJS Date Range Picker in Neutral Timezone"
description: "This article provides a solution to using and storing dates or timings with AngularJS Daterangepicker in a timezone-neutral manner."
date:   2015-11-09
---

<p class="intro"><span class="dropcap">P</span>reviously, I blogged about a way to store dates or timings using the Angular UI Bootstrap widgets in a timezone-neutral manner.</p>

In this [previous blog article]({% post_url 2015-11-06-angularui-ui-bootstrap-datepicker-and-timepicker-neutral-timezone %}), I presented a way to store dates and timings in a timezone-neutral manner using the popular [Angular UI Bootstrap widgets](https://angular-ui.github.io/bootstrap/). In order to extend the functionality of my website, I had to allow the selection of date ranges, but I was looking for a AngularJS widget that is much more user friendly than selecting 2 different dates using standard datepickers. Eventually, I stumbled upon [https://github.com/fragaria/angular-daterangepicker](https://github.com/fragaria/angular-daterangepicker) which boasts sufficiently powerful features, though lacking in timezone support again.

### Why do we need timezone support? ###

To cut the chase short, I will suggest that you check out my [previous post]({% post_url 2015-11-06-angularui-ui-bootstrap-datepicker-and-timepicker-neutral-timezone %}) about the need for timezone support.

### How to convert the dates to a neutral timezone using the angular-daterangepicker widget? ###

If you read my [previous post]({% post_url 2015-11-06-angularui-ui-bootstrap-datepicker-and-timepicker-neutral-timezone %}), you will notice that the code in that article could probably serve the same function here.
However, there's a caveat.
The `angular-daterangepicker` widget stores dates (or timings) internally as MomentJS objects by utilizing the [MomentJS library](http://momentjs.com/), and MomentJS objects are different from standard Date objects.

So below here is the revised code to support the `angular-daterangepicker` widget. The part that is revised is mainly the parser function.

{% highlight javascript %}
angular.module('app')
  .directive('daterangepickerNeutralTimezone', function() {
    return {
      restrict: 'A',
      priority: 1,
      require: 'ngModel',
      link: function (scope, element, attrs, ctrl) {
        ctrl.$formatters.push(function (value) {
          if (value.startDate && value.endDate) {
            var startDate = new Date(Date.parse(value.startDate));
            startDate = new Date(startDate.getTime() + (60000 * startDate.getTimezoneOffset()));
            var endDate = new Date(Date.parse(value.endDate));
            endDate = new Date(endDate.getTime() + (60000 * endDate.getTimezoneOffset()));
            return {startDate: startDate, endDate: endDate};
          } else {
            return value;
          }
        });

        ctrl.$parsers.push(function (value) {
          if (value.startDate && value.endDate) {
            var startDate = value.startDate.add(60000 * value.startDate.utcOffset());
            var endDate = value.endDate.add(60000 * value.endDate.utcOffset());
            return {startDate: startDate, endDate: endDate};
          } else {
            return value;
          }
        });
      }
    };
  });
{% endhighlight %}

And again, you would use the directive together with the `angular-daterangepicker` widget like this:

{% highlight html %}
<input date-range-picker class="form-control date-picker" type="text" ng-model="range" min="minDate" daterangepicker-neutral-timezone />
{% endhighlight %}

Similarly, you can then display the stored dates using the AngularJS `date` filter adjusted to the UTC timezone.

{% highlight javascript %}
{% raw %}
{{ range.startDate | date:null:'UTC' }} - {{ range.endDate | date:null:'UTC' }}
{% endraw %}
{% endhighlight %}
