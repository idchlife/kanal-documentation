# Routing

{% info NOTE %}
If you are reading this first time, please don't forget to read chapter about Router behaviour,
because without that information you most likely will stumble on some unexpected behaviour, without
knowing first how routes are processed.
{% end %}

Routing in Kanal is written using handy DSL like this:

```ruby
...
core.router.configure do
  on :body, contains: "my name is" do
    respond do
      body "Hey good to know!"
    end
  end

  on :body, contains: "what is the weather today?" do
    respond do
      weather = core.services.get(:weather_service).get_weather

      body "Todays weather is: #{weather}"
    end
  end
end
...
```

{% info NOTE %}
If you are wondering what is the core.services and why is there core available,
please check more in `respond block` documentation and page about Services.
{% end %}

# Router.configure and DSL

When you pass block of code into the `router.configure` method, you
are presented with a DSL methods that helps you to write your routes.

At the moment these DSL methods are:

- on

## on

This method accepts 3 arguments. Well, 2 arguments and a block

- First argument is a name of a condition pack, a Symbol
- Second argument is a name of a condition. It can be written in 2 ways, without argument or with argument,
  depends on conditions (whether it accepts argument)

```ruby
...
# When condition does not require argument
on :condition_pack_name, :condition_name do

end

# When condition requires argument
on :condition_pack_name, condition_name: you_argument do

end
...
```

# Default response

Default response is a vital part of a router and it should be provided.

Without default response - Kanal won't start, its router will raise an exception asking you
to provide default response.

You can specify default response like this:

```ruby
core.router.default_response do
  # Here you can write code like you would normally write code in respond block.
  # With access to input, core and setting output parameters
end
```

{% info NOTE %}
Default response is required for router to operate, because if not suitable route
found, router will refer to the provided default response.

Usually default response looks like "Unfortunately, I did not quite get this. I can understand these commands: ... ... ..."
{% end %}

# Nested routes

You can have nested routes. It means you can write something like this:

```ruby
core.router.configure do
  on :body, starts_with: "tell me" do
    on :body, ends_with: "news" do
      respond do
        body "News today as good as they get!"
      end
    end

    on :body, ends_with: "joke" do
      body core.services.get(:jokes_service).random_joke
    end
  end

  on :body, contains: "what is the weather today?" do
    respond do
      weather = core.services.get(:weather_service).get_weather

      body "Todays weather is: #{weather}"
    end
  end
end
```

You can have as many nesting layers as you want.

{% warn ATTENTION %}
Beware, when you are using nested routes: if route contains at least 1 nested route, you can no
longer use `respond` block inside.

Your route can contains one of the 2: nested route(s) **OR** respond block

If you wonder about the default response in the route, if no nested routes were suitable with their conditions,
please read further on this page how router machinery works and how to control your input flow in routes.
{% end %}

# Respond block

In your route, you can specify `respond block`

Like this:

```ruby
...
on :body, ends_with: "news" do
  # This is the respond block we are talking about
  respond do
    body "News today as good as they get!"
  end
end
...
```

## Setting output parameters

Respond block allows you to set registered in output parameters. Meaning if you registered parameter
`block`, it will be available as DSL method `block` inside. You specify parameter value by using method syntax,
but not variable syntax.

```ruby
# Wrong
body = "Somebody"

# Right
body "Somebody"
```

## Accessing input, core

Kanal provides you with `input` and `core` inside respond block.

Using `input`, you can have additional checks and validations inside and give response to user accordingly.

Like this:

```ruby
on :body, ends_with: "news" do
  # This is the respond block we are talking about
  respond do
    user_said = input.body

    body "You said: #{user_said} and my response is: News today as good as they get!"
  end
end
```

Why there is `core` available? So you can access anything that can core provide.

For example, if you registered service named :weather_provider, you can acces it there and use it

```ruby
on :body, contains: "weather" do
  # This is the respond block we are talking about
  respond do
    weather = core.services.get(:weather_provider).get_current_weather

    body "Current weather is: #{weather}"
  end
end
```

{% info NOTE %}
Please be aware of examples like this weather example, that usually when someone wants to know
weather at their location, it is required to get users location in one way or another.

It is of course possible, if user sends location via some bot or writes location by hand (lat/long or name of city),
but this is topic for another documentation page about integrations with bots etc.

For the sake of example let's pretend that user wants to know weather at some kind of default location for everyone,
without providing the location manually.
{% end %}

# Router expected behaviour and how it operates

Basically router stores routes in a tree. Well, for some people it will be obvious,
looking at how you write your routes.

What is a tree? Imagine, there is a root with branches coming out of it, and then there are branches coming out of branches etc. etc.

When input is provided to the method `core.router.create_output_for_input` router goes from root to all the branches (routes),
test input against conditions in those routes and finally finds a suitable route or refers to the default response.

Consider this router configuration:

```ruby
core.router.default_response do
  body "I can't understand you!"
end

core.router.configure do
  on :body, contains: "hey" do
    respond do
      body "Hello to you!"
    end
  end

  on :body, contains: "tell me" do
    on :body, contains: "weather" do
      respond do
        body "Weather is fine!"
      end
    end

    on :body, contains: "news" do
      respond do
        body "News are great!"
      end
    end
  end
end
```

Now, let's imagine this code with comments, showing which input will get which output

```ruby
input = core.create_input
input.body = "Hello"

output = core.router.create_output_for_input input

# output.body will be "I can't understand you!"

input = core.create_input
input.body = "tell me weather"

output = core.router.create_output_for_input input

# output.body will be "Weather is fine!"

input = core.create_input
input.body = "tell me something"

output = core.router.create_output_for_input input

# output.body will be "I can't understand you!"
```

So far so good. But what happens, if there are 2 routes sitting next to
each other that have suitable condition for our input? But, only second route contains suitable condition inside.

## When 2 adjacent routes accepts same input (but inner routes do not)

Let's take this example configuration:

```ruby
core.router.default_response do
  body "I can't understand you!"
end

core.router.configure do
  on :body, contains: "tell me" do
    on :body, contains: "weather" do
      respond do
        body "Weather is fine!"
      end
    end
  end

  on :body, contains: "never" do
    on :body, contains: "forever" do
      respond do
        body "This is really interesting"
      end
    end
  end
end
```

This doesn't seem so bad, we see different conditions for these 2 routes, but...

Imagine this code with input:

```ruby
input = core.create_input
input.body = "tell me never"

output = core.router.create_output_for_input input

# output.body will be "This is really interesting"
```

As you can see, router successfully found the route for this input!

How did it find it?

Well, first it tried route with condition `contains: "weather"`. It got inside this route,
and tried inner route for `contains: weather` and got condition check failed.

What happened next? Router got back up in the routes branches and started checking next routes!

And next route and inner route had condition that successfully validated our input!

Remember this behaviour, router does not stop in any inner route branches if it does not find
suitable route. It traverses back up and tests all other adjacent branches.

{% info NOTE %}
Remember, routes are traversed and tested for condition in the order they were written in the
configure block. 

Meaning first defined - first tested for.
{% end %}