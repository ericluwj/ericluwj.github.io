---
layout: post
title:  "Authentication Popup Modals or Notifications in AngularJS"
description: "For a long time, I had this big question in my mind: Should I use spawn or execFile? I am actually referring to the 2 Node.js functions require('child_process').spawn() and require('child_process').execFile()."
date:   2015-12-02
---

<p class="intro"><span class="dropcap">I</span> had previously implemented a simple authentication system tied in with UI Router for my own AngularJS app. It can either pop up an optional or compulsory login modal depending on the UI Router $state settings.</p>

I believe this is a better user experience by prompting the user to login while having a bit of view at the back, instead of just redirecting the user away to a login page. Below are some code snippets of how this system is implemented.

{% highlight javascript %}
function authenticateState(next) {
  /*
  Kinds of authentication action:
  1) Show optional login modal if unauthenticated (authAction=promptIfUnauthenticated)
  2) Show compulsory login modal if unauthenticated (authAction=loginIfUnauthenticated)
  3) Redirect if unauthenticated  (authAction=redirectIfUnauthenticated)
  4) Redirect if authenticated (authAction=redirectIfAuthenticated)
  5) Do nothing (authAction=undefined)
  */
  Auth.isLoggedInAsync(function(loggedIn) {
    if (next.data) {
      if (loggedIn && next.data.authAction === 'redirectIfAuthenticated') {
        $state.go('main');
      } else if (!loggedIn && next.data.authAction === 'redirectIfUnauthenticated') {
        $location.path('/login');
      } else if (!loggedIn && next.data.authAction === 'loginIfUnauthenticated') {
        var modalInstance = $modal.open({
          templateUrl: 'components/application/application.modal.login.html',
          controller: 'ApplicationLoginCtrl',
          size: 'sm',
          backdrop: 'static',
          keyboard: false
        });
        modalInstance.result.then(function (user) {
          $state.forceReload();
        }, function() {
          $state.go('main');
        });
      } else if (!loggedIn && next.data.authAction === 'promptIfUnauthenticated') {
        var modalInstance = $modal.open({
          templateUrl: 'components/application/application.modal.login.html',
          controller: 'ApplicationLoginCtrl',
          size: 'sm'
        });
        modalInstance.result.then(function (user) {
          $state.forceReload();
        });
      } else if (loggedIn && Auth.getCurrentUser().role === 'hirer' && Auth.getCurrentUser().generated) {
        var modalInstance = $modal.open({
          templateUrl: 'components/application/application.modal.claim.html',
          controller: 'ApplicationClaimCtrl',
          size: 'sm',
          backdrop: 'static',
          keyboard: false
        });
        modalInstance.result.then(function (user) {
          $state.forceReload();
        }, function() {
          $state.go('main');
        });
      }
    }
  });
}

// Redirect to login if route requires auth and you're not logged in
$rootScope.$on('$stateChangeStart', function (event, toState) {
  authenticateState(toState);
});
{% endhighlight %}

I don't think I am going to go through my code snippet thoroughly, but I have set a few kinds of authorization actions for different pages/states. Some states will pop up modals to prompt for or require login action.

I thought that this was perfect and good to go, but more issues arose later. This is the first issue:

> The modals were hard to keep track. I had a instance of multiple modals opening together at the same time.

Let's say if I first landed on a page where the authentication modal was opened. Then I navigated to another page on which the authentication modal will not be opened, but the authentication modal would still be left open, so the navigation did not have any instruction to close the previously opened modal. There were basically 2 possible solutions for this issue. One way was simply to abstract the $modal service so that we could keep track of modals opened possibly from anywhere in the application. The other way was simply to add a cheap hack to close all modal windows at the start of a state change with the following code addition.

{% highlight javascript %}
// Redirect to login if route requires auth and you're not logged in
$rootScope.$on('$stateChangeStart', function (event, toState) {
  // clear all modals (if existing) first on new page
  $('.modal-content > .ng-scope').each(function() {
    try {
      $(this).scope().$dismiss();
    } catch(_) {}
  });

  authenticateState(toState);
});
{% endhighlight %}

Both solutions should work fine, but I implemented the hack to save time. Then, another issue occurred while we had to pre-render our SPA application pages. While our headless browser does its snapshots of our pages, guess what?

>The authentication modals popped up because the "user" was not authenticated, and the opened modals got taken in the snapshot.

As the snapshots are like pre-rendered AngularJS `index.html` files to be served, the applications run from the snapshots naturally did not understood that a modal was already opened. (If you are not sure as to what I am talking about, maybe you can check out [this article]({% post_url 2015-11-17-seo-for-angularjs-on-s3 %}).) You would notice that keeping track of opened modals would not help at all. The way I solved it was, to disable such authentication modals from appearing for search bots and snapshot PhantomJS bot.

{% highlight javascript %}
function isSearchBot() {
  return /bot|googlebot|google|Google-StructuredDataTestingTool|facebot|facebookexternalhit|twitterbot|pinterest|crawler|spider|robot|crawling/i.test(navigator.userAgent);
}

function isPhantomBot() {
  return /PhantomJS/i.test(navigator.userAgent);
}

function authenticateState(next) {
  /*
  Kinds of authentication action:
  1) Show optional login modal if unauthenticated (authAction=promptIfUnauthenticated)
  2) Show compulsory login modal if unauthenticated (authAction=loginIfUnauthenticated)
  3) Redirect if unauthenticated  (authAction=redirectIfUnauthenticated)
  4) Redirect if authenticated (authAction=redirectIfAuthenticated)
  5) Do nothing (authAction=undefined)
  */
  Auth.isLoggedInAsync(function(loggedIn) {
    if (next.data) {
      if (loggedIn && next.data.authAction === 'redirectIfAuthenticated') {
        $state.go('main');
      } else if (!loggedIn && next.data.authAction === 'redirectIfUnauthenticated') {
        $location.path('/login');
      } else if (!loggedIn && next.data.authAction === 'loginIfUnauthenticated') {
        if (!isSearchBot() && !isPhantomBot()) {
          var modalInstance = $modal.open({
            templateUrl: 'components/application/application.modal.login.html',
            controller: 'ApplicationLoginCtrl',
            size: 'sm',
            backdrop: 'static',
            keyboard: false
          });
          modalInstance.result.then(function (user) {
            $state.forceReload();
          }, function() {
            $state.go('main');
          });
        }
      } else if (!loggedIn && next.data.authAction === 'promptIfUnauthenticated') {
        if (!isSearchBot() && !isPhantomBot()) {
          var modalInstance = $modal.open({
            templateUrl: 'components/application/application.modal.login.html',
            controller: 'ApplicationLoginCtrl',
            size: 'sm'
          });
          modalInstance.result.then(function (user) {
            $state.forceReload();
          });
        }
      } else if (loggedIn && Auth.getCurrentUser().role === 'hirer' && Auth.getCurrentUser().generated) {
        if (!isSearchBot() && !isPhantomBot()) {
          var modalInstance = $modal.open({
            templateUrl: 'components/application/application.modal.claim.html',
            controller: 'ApplicationClaimCtrl',
            size: 'sm',
            backdrop: 'static',
            keyboard: false
          });
          modalInstance.result.then(function (user) {
            $state.forceReload();
          }, function() {
            $state.go('main');
          });
        }
      }
    }
  });
}

// Redirect to login if route requires auth and you're not logged in
$rootScope.$on('$stateChangeStart', function (event, toState) {
  // clear all modals (if existing) first on new page
  $('.modal-content > .ng-scope').each(function() {
    try {
      $(this).scope().$dismiss();
    } catch(_) {}
  });

  authenticateState(toState);
});
{% endhighlight %}

I know the code is a bit messy without proper abstraction, but then, I was merely trying to share the principles and solutions.