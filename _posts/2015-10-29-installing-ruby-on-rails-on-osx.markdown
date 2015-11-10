---
layout: post
title:  "Installing Ruby on Rails on OS X"
description: "Sometimes, I just can't help it but lose faith in Ruby on Rails. Sure, Ruby on Rails provides a really great framework, but I have never been excited about its mediocre processing speed and of course the installation nightmare it causes on Mac OS X."
date:   2015-10-29

redirect_from: "/blog/installing-ruby-on-rails-on-osx/"
---

<p class="intro"><span class="dropcap">S</span>ometimes, I just can't help it but lose faith in Ruby on Rails. Sure, Ruby on Rails provides a really great framework, but I have never been excited about its mediocre processing speed and of course the installation nightmare it causes on Mac OS X.</p>

While trying to install Ruby on Rails with the supposedly basic command `gem install rails` on my El Capitan OS X, I encountered this error:

	-----
	libxml2 is missing.  Please locate mkmf.log to investigate how it is failing.
	-----
	*** extconf.rb failed ***
	Could not create Makefile due to some reason, probably lack of necessary
	libraries and/or headers.  Check the mkmf.log file for more details.  You may
	need configuration options.

	Provided configuration options:
		--with-opt-dir
		--without-opt-dir
		--with-opt-include
		--without-opt-include=${opt-dir}/include
		--with-opt-lib
		--without-opt-lib=${opt-dir}/lib
		--with-make-prog
		--without-make-prog
		--srcdir=.
		--curdir
		--ruby=/Users/ericluwj/.rbenv/versions/2.2.3/bin/$(RUBY_BASE_NAME)
		--help
		--clean
		--use-system-libraries
		--enable-static
		--disable-static
		--with-zlib-dir
		--without-zlib-dir
		--with-zlib-include
		--without-zlib-include=${zlib-dir}/include
		--with-zlib-lib
		--without-zlib-lib=${zlib-dir}/lib
		--enable-cross-build
		--disable-cross-build
		--with-xml2lib
		--without-xml2lib
		--with-libxml2lib
		--without-libxml2lib

	extconf failed, exit code 1

	Gem files will remain installed in /Users/ericluwj/.rbenv/versions/2.2.3/lib/ruby/gems/2.2.0/gems/nokogiri-1.6.6.2 for inspection.
	Results logged to /Users/ericluwj/.rbenv/versions/2.2.3/lib/ruby/gems/2.2.0/extensions/x86_64-darwin-15/2.2.0-static/nokogiri-1.6.6.2/gem_make.out

Mind you, I was simply following the instructions provided on the [Ruby on Rails download site](http://rubyonrails.org/download/). With OS X being such a popular development environment, it is really disappointing to see how **challenging** it can be to actually install Ruby on Rails. If I am an average Mac developer, and I'm facing these issues, I'm sure many other Mac developers would have faced or would be facing this issue, so I have decided to document the **right** way to install Ruby on Rails.

Just as a sidenote, the issue stemmed mainly from libxml2, a dependency of the Nokogiri gem, which is also a dependency of the Rails gem. The OS X version of libxml simply does not work well with the compiling of libraries for Nokogiri. I even tried using the `brew` version of libxml2, but it does not work either for me. Even though Nokogiri now bundles its own version of libxml2, the issue still persists. What an irony!

I would say the best solution is still to depend on the updated version of libxml2 that is bundled with Xcode, rather than to depend on Rails or Nokogiri to solve this whole mess.

##Installing Ruby on Rails##

If you are installing Ruby on Rails, I would assume that you are a developer on OS X and you would have Xcode. Do go ahead and ensure that you have agreed to the Xcode license in order to use the libraries bundled with Xcode.

First, install Nokogiri gem by linking to the libxml2 library bundled with Xcode.

**Mac OS X 10.9 Mavericks**

	gem install nokogiri -- --with-xml2-include=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.9.sdk/usr/include/libxml2

**Mac OS X 10.10 Yosemite**

	gem install nokogiri -- --with-xml2-include=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/usr/include/libxml2 --use-system-libraries

**Mac OS X 10.11 El Capitan**

	gem install nokogiri -- --with-xml2-include=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk/usr/include/libxml2 --use-system-libraries

*If you do still experience a missing libxml2 issues, just check the xml2 include directory for its exact location.*

Second, simply install Rails gem as per normal.

	gem install rails

Finally, you have managed to install Ruby on Rails smoothly.