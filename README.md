Sansom
===

Scientific, philosophical, abstract web 'picowork' named after Sansom street in Philly, where it was made.

Philosophy
-

***A piece of software should not limit you to one way of thinking.***

You can write a `Sansomable` for each logical unit of your API, but you also don't have to.

You can also mount existing Rails/Sinatra/Rack apps in your `Sansomable`. But you also don't have to.

You can write one `Sansomable` for your entire API.

Fuck it.

***A tool should do one thing, and do it well. (Unix philosophy)***

A web framework is, fundamentally, a tool to connect code to a URL's path.

A web framework doesn't provide an ORM, template rendering, shortcuts, nor security patches. 

***A web framework shall remain a framework***

No single tool should get so powerful that it can overcome its master. Rails and Sinatra have been doing this: modifying the language beyond recognition. Rails has activerecord and Sinatra's blocks aren't really blocks.

Installation
-

`gem install sansom`

General Usage
-
Traditional approach:

    # config.ru
    
    require "sansom"
    
    s = Sansom.new
    # define routes on s
    run s
    
One-file approach:

    # app.rb

	require "sansom"

    s = Sansom.new
    # define routes on s
    s.start
    
They're basically the same, except the rack server evaluates config.ru in its own context. The config.ru approach allows for the config to be separated from the application code.

Writing your own traditional-style webapp
-

Writing a one-file webapp is as simple as creating a `Sansomable`, defining routes on it, and calling start on it.

####There is more footwork for a traditional-style webapp:

Sansom is defined like this:

    Sansom = Class.new Object
    Sansom.send :include, Sansomable

So you'll want your app to either `include Sansomable` or be a subclass of `Sansom`, so that a basic declaration looks like this.

	# myapi.rb
	
	require "sansom"
	
	class MyAPI
	  include Sansomable
	  def template
	  	# define routes here
	  end
	end
    
And your `config.ru` file

    # config.ru
    
    require "./myapi"
    
    run MyAPI.new
    
Defining Routes
-
Routes can be defined like so:

    s = Sansom.new
    s.get "/" do |r| # r is a Rack::Request
	  [200, {}, ["Return a Rack response."]]
    end

You can replace `get` with any http verb. Or `map`, if you want to map a subsansom. Let's say you've written a new version of your api. No problem:
    
    # app.rb
    
    require "sansom"
    
    s = Sansom.new
    s.map "/v1", MyAPI.new
    s.map "/v2", MyNewAPI.new
    s.start
    
Sansom blocks vs Sinatra blocks
-

Sansom blocks remain blocks: When a route is mapped, the same block you use is called when a route is matched. It's the same object every time.

Sinatra blocks become methods behind the scenes. When a route is matched, Sinatra looks up the method and calls it.

Sinatra's mechanism allows for the use of `return` inside blocks. Sansom doesn't do this, so you must use the `next` directive in the same way you'd use return.

Before filters
-

You can write before filters to try to preëmpt request processing. If the block returns a valid response, the request is preëmpted & it returns that response.

    # app.rb
    
    require "sansom"
    
    s = Sansom.new
    s.before do |r|
      next [200, {}, ["Preëmpted."]] if some_condition
    end
    
You could use this for request statistics, caching, auth, etc.

After filters
-

You can also write after filters to tie up the loose ends of a response. If they return a valid response, that response is used instead of the response from a route. After blocks are not called if a before filter was ever called.

    # app.rb
    
    require "sansom"
    
    s = Sansom.new
    s.after do |req,res| # req is a Rack::Request and res is the response generated by a route.
      next [200, {}, ["Postëmpted."]] if some_condition
    end

Errors
-

Error blocks allow for the app to return something parseable when an error is raised.

    require "sansom"
    require "json"
    
    s = Sansom.new
    s.error do |err, r| # err is the error, r is a Rack::Request
      [500, {"yo" => "shit"}, [{ :message => err.message }.to_json]]
    end
    
There is also a unique error 404 handler:

    require "sansom"
    require "json"
    
    s = Sansom.new
    s.not_found do |r| # r is a Rack::Request
      [404, {"yo" => "shit"}, [{ :message => "not found" }.to_json]]
    end

Matching
-

`Sansom` uses trees to match routes. It follows a certain set of rules:

  - Wildcard routes can't have any siblings
  - A matching order is enforced:
  	1. The route matching the path and verb
  	2. The first Subsansom that matches the route & verb
  	3. The first mounted non-`Sansom` rack app matching the route
  	
Some examples of routes Sansom recognizes:  
	`/path/to/resource` - Standard path  
	`/users/:id/show` - Parameterized "wildcard" path
	`/services/show.<format>` - Semi-wilcard

Notes
-

- `Sansom` does not pollute _any_ `Object` methods, including `initialize`
- No regexes are used in route matching, the're 3x slower than raw string manipulation. They are used in mapping semiwildcards.
- `Sansom` is under **400** lines of code at the time of writing. This includes
	* Rack conformity & the DSL (`sansom.rb`) (about 100 lines)
	* Custom tree-based routing (`sanom/pine.rb`) (

Speed
-

Well, that's great and all, but how fast is "hello world" example in comparision to Rack or Sinatra?

Rack: **11ms**<br />
Sansom: **14ms**\*<br />
Sinatra: **28ms**<br />
Rails: **34ms****

(results are measured locally using Puma and are rounded down)

Hey [Konstantine](https://github.com/rkh), *put that in your pipe and smoke it*.

\* Uncached. If a tree lookup is cached, it will be pretty much as fast as Rack.
\** Rails loads a rich welcome page which may contribute to its slowness

Todo
-

* Multiple return types for routes

If you have any ideas, let me know!

Contributing
-

You know the drill. But ** make sure you don't add tons and tons of code. Part of `Sansom`'s beauty is is brevity.**