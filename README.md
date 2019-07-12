Cache Crispies [![Build Status](https://travis-ci.org/codenoble/cache-crispies.svg?branch=master)](https://travis-ci.org/codenoble/cache-crispies) [![Maintainability](https://api.codeclimate.com/v1/badges/278cfda71defc0bc1d1c/maintainability)](https://codeclimate.com/github/codenoble/cache-crispies/maintainability) [![Test Coverage](https://api.codeclimate.com/v1/badges/278cfda71defc0bc1d1c/test_coverage)](https://codeclimate.com/github/codenoble/cache-crispies/test_coverage)
==============

Speedy Rails JSON serialization with built-in caching.

Why?
----

There are a lot of Rails serializers out there, but there seem to be very few these days that are well maintained and performant. The ones that are, tend to lock you into a specific standard for how to format your JSON responses. And the idea of introducing breaking API changes across the board to a mature Rails app is daunting, to say the least.

In additon, incorporating a caching layer (for performance reasons) into your serializers can be difficult unless you do it at a Rails view layer. And the serialization gems that work at the view layer tend to be slow in comparison to others. So it tends to be a one step forward one step back sort of solution.

In light of all that, this gem was built with these goals in mind:
1. Be fast
2. Support caching in as simple a way as we can
3. Support rollout without causing breaking API changes
4. Avoid the bloat that can lead to slowness and maintenance difficulties

Requirements
------------
- Ruby `2.6` _(others will likely work but are untested)_
- Rails 5 _(others may work but are untested)_
- Redis (for caching)

Features
--------
- **Fast** even without caching _(benchmarks and comparisons coming soon)_
- **Flexible** lets you serialize data any way you want it
- **Built-in Caching** _(documentation coming soon)_
- **ETags** for easy HTTP caching
- **Simlpe, Readable DSL**

Usage
-----
### A simple serializer
```ruby
class CerealSerializer < CacheCrispies::Base
  serialize :name, :brand
end
```
Put serializer files in `app/serializers/`. For instance this file should be at `app/serializers/cache_serializer.rb`.

### In your Rails controller
```ruby
class CerealsController
  def index
    cereals = Cereal.all
    cache_render CerealSerializer, cereals, custom_option: true
  end
end
```

How To...
---------
### Use a different JSON key
```ruby
serialize :is_organic, from: :organic?
```

### Nest another serializer
```ruby
serialize :incredients, with: IngredientSerializer
```

### Merge attributes from another serializer
```ruby
merge :legal_info, with: LegalInfoSerializer
```

### Coerce to another data type
```ruby
serialize :id, to: String
```
Supported data type arguments are
- `String`
- `Integer`
- `Float`
- `BigDecimal`
- `Array`
- `Hash`
- `:bool`, `:boolean`, `TrueClass`, or `FalseClass`

### Nest attributes
```ruby
nest_in :health_info do
  serialize :non_gmo
end
```
_You can nest `nest_in` blocks as deeply as you want._

### Conditionally render attributes
```ruby
show_if (model, options) => { model.low_carb? || options[:trendy] } do
  serialize :keto_certified
end
```
_You can nest `show_if` blocks as deeply as you want._

### Render custom values
```ruby
serialize :fine_print

def fine_print
  model.fine_print || options[:fine_print] || '*Contents may contain lots and lots of sugar'
end
```

### Include other data
```ruby
class CerealSerializer < CacheCrispies::Base
  serialize :page

  def page
    options[:page]
  end
end

CerealSerializer.new(cereal, page: 42).as_json
# or
cache_render CerealSerializer, cereal, page: 42
```

### Set custom JSON keys
```ruby
class CerealSerializer < CacheCrispies::Base
  def self.key
    :breakfast_cereal
  end

  def self.collection_key
    :breakfast_cereals
  end
end
```
_Note that `collection_key` is the plural of `key` by default._

### Render a serializer to a `Hash`
```ruby
CerealSerializer.new(Cereal.first, trendy: true).as_json
```


_**NOTE:** Documentation for caching is coming soon._

Tips
----
To delete all cache entries in Redis:
`redis-cli --scan --pattern "*cache-crispies*" | xargs redis-cli unlink`

License
-------
MIT

To Do List
----------
- Caching documentation
- Incorporate nested serializer's keys into the top serializers cache key
- Provide a way to handle the fact that a serializer may include a mixin or something that might have changed and should bust the cache.
- Blocks for custom attributes (if performance hit is minimal)
- through: option for delegation (if performance hit is minimal)
- Partial caching
- Rake task to clear out cache keys
