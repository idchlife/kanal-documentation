# Introduction to conditions

Conditions are basically blocks that decide whether this particular `input` is going inside this particular
route with this particular condition.

Let's take this route, for example:

```ruby
core.router.configure do
  on :body, :contains_hello do
    respond do
      body "Hello to you!"
    end
  end
end
```

The part `:body` is condition pack named :body, and part `:contains_hello` is condition named :contains_hello

To create this condition pack and condition we would write this code:

```ruby
core.add_condition_pack :body do
  add_condition :contains_hello do
    met? |input, core, argument| do
      input.body.include? "hello"
    end
  end
end
```

# Condition pack

Conditions are always stored in a pack. This way, pack serves as a namespace where your
conditions are stored.

Thanks to this, different packs can have conditions with the same name.

For example, we can have condition pack :phone with
condition :valid and can have condition pack :email with condition :valid.
Or another example - pack :video with condition max_size: and pack :image with condition max_size:

```ruby
core.add_condition_pack :condition_pack_name do
  # Here you write your conditions with handy DSL
end
```

# Conditions

## Conditions without arguments

```ruby
core.add_condition_pack :condition_pack_name do
  add_condition :condition_name do
    ...
  end
end
```

This is syntax for simple condition without argument. Meaning that the behaviour of condition
will be specified inside condition and basically cannot be affected by some kind of argument.
Examples of possible condition packs with conditions without arguments:

- :file
  - :exists
  - :mp3
  - :ogg
  - :jpg
  - :bmp

- :body
  - :has_forbidden_words
  - :empty
  - :has_emoji

## Conditions with arguments

Conditions with arguments provide the needed flexibility for specifying
argument when you are writing your routes.

This is the syntax of writing conditions with arguments:

```ruby
core.add_condition_pack :condition_pack_name do
  add_condition :condition_name do
    with_argument # This part is important

    ...
  end
end
```

As you can see, the only difference here is the DSL part `with_argument`

By writing this you mark
condition as condition with argument.

Thanks to this you will be able to write conditions in your routes like this:

```ruby
...
core.router.configure do
  on :body, contains: "Hey!" do
    ...
  end
end
...
```

As you can see, we pass name of condition and it's argument as it was simple named parameter.

## met? block arguments

Met block is the block which will actually check whether conditions are met for arguments provided.

To complete the previous example of conditions with arguments -
here how condition with :body and :contains can be written

```ruby
core.add_condition_pack :body
  add_condition :contains do
    met? do |input, core, argument| # 3 arguments provided to the block
      input.body.include? argument
    end
  end
end
```

As you probably read already in this documentation, met block is provided with 3 arguments:

- input
- core
- argument

### Input

This is the input you insert into the routers method .create_output_for_input

This input will be provided to the each route and there it will be checked against the
saved into the condition met block.

This is your main source of the checks, validations etc.

{% info NOTE %}
It is fare to say that you can have conditions with met blocks not utilizing input argument at all.
For example something like pack :time and condition :evening. It will simply check the current time
{% end %}

### Core

Core provided as a source of the available parts of the core, possibility to get services
(more about them on Services documentation page), check whether some plugin registered and so on.

If you need to utilize the core as argument in met block, basically you know what you are doing.

You are getting some service, or checking if this bot has some plugin registered for debug purpose.

### Argument

This one is provided when you use `conditions WITH arguments`. When you are using
conditions without arguments, this block would be just nil.

{% info NOTE %}
You can pass as argument anything you want. It is not limited to any primitive data type.
You can even pass there instance of a class, or a list, or lambda etc.
{% end %}

# Conclusion

Conditions allow you to direct input in your routes to a different places.

They are a powerful way to avoid writing numerous if/else inside one block of
route and separate them into different routes.

By creating condition pack and conditions within you basically create a reusable
code which can be utilized in multiple routes and even projects.