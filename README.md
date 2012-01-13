This project demonstrates a problem I'm having with the trinidad_resque_extension.

## The Problem

Here's how i created this app:

	$ rails new blog
	$ cd blog
	$ edit Gemfile
	
Add these to my Gemfile:
	
	gem "trinidad", "1.2.3"
	gem "trinidad_jars", "1.0.1"
	gem "trinidad_resque_extension", "0.1.0"

Back to the console:

	$ bundle install
	$ touch config/trinidad.yml
	$ edit trinidad.yml
	
Add this to the file:

	extensions:
	  resque:
   	    queues: critical, normal, low
        count:  1
        path: 'lib/workers'
        redis_host: 'localhost:6379'
        
Back to the console:

	$ mkdir lib/workers
	
In a seperate console, start redis:

	$ redis-server
	
Back to the original console, start trinidad:

	$ bundle exec trindad
	NOTE: Gem::GemPathSearcher#initialize is deprecated with no replacement. It will be removed on or after 2011-10-01.
	Gem::GemPathSearcher#initialize called from /Users/jkutner/.rvm/gems/jruby-1.6.5/gems/trinidad_resque_extension-0.1.0/lib/trinidad_resque_extension.rb:51.
	NOTE: Gem::GemPathSearcher#find is deprecated with no replacement. It will be removed on or after 2011-10-01.
	Gem::GemPathSearcher#find called from /Users/jkutner/.rvm/gems/jruby-1.6.5/gems/trinidad_resque_extension-0.1.0/lib/trinidad_resque_extension.rb:51.
	Jan 12, 2012 3:47:45 PM org.apache.coyote.AbstractProtocolHandler init
	INFO: Initializing ProtocolHandler ["http-bio-3000"]
	Jan 12, 2012 3:47:45 PM org.apache.catalina.core.StandardService startInternal
	INFO: Starting service Tomcat
	Jan 12, 2012 3:47:45 PM org.apache.catalina.core.StandardEngine startInternal
	INFO: Starting Servlet Engine: Apache Tomcat/7.0.11
	2012-01-12 15:47:45 INFO: No global web.xml found
	2012-01-12 15:47:46 INFO: Info: received max runtimes = 5
	2012-01-12 15:47:46 INFO: jruby 1.6.5 (ruby-1.8.7-p330) (2011-10-25 9dcd388) (Java HotSpot(TM) 64-Bit Server VM 1.6.0_29) [darwin-x86_64-java]
	2012-01-12 15:47:46 INFO: Info: using runtime pool timeout of 30 seconds
	2012-01-12 15:47:46 INFO: Info: received min runtimes = 1
	2012-01-12 15:47:46 INFO: Info: received max runtimes = 5
	rake aborted!
	undefined method `require_dependency' for #<Blog::Application:0x78abd604>
	
	Tasks: TOP => resque:work => resque:preload
	(See full trace by running task with --trace)
	2012-01-12 15:47:59 INFO: Info: add application to the pool. size now = 1
	Invalid log level , using default: INFO
	2012-01-12 15:47:59 INFO: The start() method was called on component [Realm[Simple]] after start() had already been called. The second call will be ignored.
	2012-01-12 15:47:59 INFO: No global web.xml found
	2012-01-12 15:47:59 INFO: jruby 1.6.5 (ruby-1.8.7-p330) (2011-10-25 9dcd388) (Java HotSpot(TM) 64-Bit Server VM 1.6.0_29) [darwin-x86_64-java]
	:public is no longer used to avoid overloading Module#public, use :public_folder instead
		from /Users/jkutner/.rvm/gems/jruby-1.6.5/gems/resque-1.19.0/lib/resque/server.rb:12:in `Server'
	2012-01-12 15:48:00 INFO: Starting ProtocolHandler ["http-bio-3000"]
	
The key part of that log is this:

	undefined method `require_dependency' for #<Blog::Application:0x78abd604>
		
Which is generated by the trinidad_resque_extension when it tries to run the
resque:work task.  If I set RAKEOPT=--trace then i can get the full trace:

	undefined method `require_dependency' for #<Blog::Application:0x142436ac>
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/railties-3.1.1/lib/rails/engine.rb:417:in `eager_load!'
	org/jruby/RubyArray.java:1612:in `each'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/railties-3.1.1/lib/rails/engine.rb:416:in `eager_load!'
	org/jruby/RubyArray.java:1612:in `each'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/railties-3.1.1/lib/rails/engine.rb:414:in `eager_load!'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/resque-1.19.0/lib/resque/tasks.rb:47:in `(root)'
	org/jruby/RubyProc.java:270:in `call'
	org/jruby/RubyProc.java:220:in `call'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/task.rb:205:in `execute'
	org/jruby/RubyArray.java:1612:in `each'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/task.rb:200:in `execute'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/task.rb:158:in `invoke_with_call_chain'
	/Users/jkutner/.rvm/rubies/jruby-1.6.5/lib/ruby/1.8/monitor.rb:191:in `mon_synchronize'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/task.rb:151:in `invoke_with_call_chain'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/task.rb:176:in `invoke_prerequisites'
	org/jruby/RubyArray.java:1612:in `each'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/task.rb:174:in `invoke_prerequisites'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/task.rb:157:in `invoke_with_call_chain'
	/Users/jkutner/.rvm/rubies/jruby-1.6.5/lib/ruby/1.8/monitor.rb:191:in `mon_synchronize'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/task.rb:151:in `invoke_with_call_chain'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/task.rb:144:in `invoke'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/application.rb:116:in `invoke_task'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/application.rb:94:in `top_level'
	org/jruby/RubyArray.java:1612:in `each'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/application.rb:94:in `top_level'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/application.rb:133:in `standard_exception_handling'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/application.rb:88:in `top_level'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/application.rb:66:in `run'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/application.rb:133:in `standard_exception_handling'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/lib/rake/application.rb:63:in `run'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/gems/rake-0.9.2.2/bin/rake:33:in `(root)'
	org/jruby/RubyKernel.java:1063:in `load'
	/Users/jkutner/.rvm/gems/jruby-1.6.5/bin/rake:19:in `(root)'

Ultimately, the workers aren't started.  

## Hack Time

The weirdest part, is that I can get it working by including Warbler in my project 
(I found this out by accident).

You can find these changes in the `warbler` branch of this repo.

First, I add Warbler to the Gemfile:

	gem "warbler"
	
Then I create a `lib/tasks/warbler.rake` file and put this in it:

	require 'bundler'
	Warbler::Task.new(:warble)
	
Now, when I start Trinidad the workers start up just fine!

	$ bundle exec trinidad
	NOTE: Gem::GemPathSearcher#initialize is deprecated with no replacement. It will be removed on or after 2011-10-01.
	Gem::GemPathSearcher#initialize called from /Users/jkutner/.rvm/gems/jruby-1.6.5/gems/trinidad_resque_extension-0.1.0/lib/trinidad_resque_extension.rb:51.
	NOTE: Gem::GemPathSearcher#find is deprecated with no replacement. It will be removed on or after 2011-10-01.
	Gem::GemPathSearcher#find called from /Users/jkutner/.rvm/gems/jruby-1.6.5/gems/trinidad_resque_extension-0.1.0/lib/trinidad_resque_extension.rb:51.
	Jan 12, 2012 4:01:11 PM org.apache.coyote.AbstractProtocolHandler init
	INFO: Initializing ProtocolHandler ["http-bio-3000"]
	Jan 12, 2012 4:01:11 PM org.apache.catalina.core.StandardService startInternal
	INFO: Starting service Tomcat
	Jan 12, 2012 4:01:11 PM org.apache.catalina.core.StandardEngine startInternal
	INFO: Starting Servlet Engine: Apache Tomcat/7.0.11
	2012-01-12 16:01:11 INFO: No global web.xml found
	2012-01-12 16:01:13 INFO: Info: received max runtimes = 5
	2012-01-12 16:01:13 INFO: jruby 1.6.5 (ruby-1.8.7-p330) (2011-10-25 9dcd388) (Java HotSpot(TM) 64-Bit Server VM 1.6.0_29) [darwin-x86_64-java]
	2012-01-12 16:01:13 INFO: Info: using runtime pool timeout of 30 seconds
	2012-01-12 16:01:13 INFO: Info: received min runtimes = 1
	2012-01-12 16:01:13 INFO: Info: received max runtimes = 5
	2012-01-12 16:01:30 INFO: Info: add application to the pool. size now = 1
	Invalid log level , using default: INFO
	2012-01-12 16:01:30 INFO: The start() method was called on component [Realm[Simple]] after start() had already been called. The second call will be ignored.
	2012-01-12 16:01:30 INFO: No global web.xml found
	2012-01-12 16:01:31 INFO: jruby 1.6.5 (ruby-1.8.7-p330) (2011-10-25 9dcd388) (Java HotSpot(TM) 64-Bit Server VM 1.6.0_29) [darwin-x86_64-java]
	:public is no longer used to avoid overloading Module#public, use :public_folder instead
		from /Users/jkutner/.rvm/gems/jruby-1.6.5/gems/resque-1.19.0/lib/resque/server.rb:12:in `Server'
	2012-01-12 16:01:33 INFO: Starting ProtocolHandler ["http-bio-3000"]
	
The best I can figure is that Warbler is loading something into the path that Rake
uses when it starts up.  But I'm pretty clueless after that.