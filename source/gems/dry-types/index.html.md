---
title: Introduction
layout: gem-single
type: gem
name: dry-types
sections:
  - including-types
  - built-in-types
  - strict
  - optional-values
  - default-values
  - sum
  - constraints
  - hash-schemas
  - array-with-member
  - enum
  - custom-types
---

`dry-types` is a simple and extendable type system for Ruby; useful for value coercions, applying constraints, defining complex structs or value objects and more. It was created as a successor to [Virtus](https://github.com/solnic/virtus).

### Example usage

```ruby
require 'dry-types'
require 'dry-struct'

module Types
  include Dry::Types.module
end

class User < Dry::Struct
  attribute :name, Types::String
  attribute :age,  Types::Int
end

User.new(name: 'Bob', age: 35)
# => #<User name="Bob" age=35>
```

See [Built-in Types](/gems/dry-types/built-in-types/) for a full list of available types.

By themselves, the basic type definitions like `Types::String` and `Types::Int` don't do anything except provide documentation about type an attribute is expected to have. However, there are many more advanced possibilities:

- 'Strict' types will raise an error if passed an attribute of the wrong type:

```ruby
class User < Dry::Struct
  attribute :name, Types::Strict::String
  attribute :age,  Types::Strict::Int
end

User.new(name: 'Bob', age: '18')
# => Dry::Struct::Error: [User.new] "18" (String) has invalid type for :age
```

- 'Coercible' types will attempt to convert an attribute to the correct class
  using Ruby's inbuilt coercion methods:

```ruby
class User < Dry::Struct
  attribute :name, Types::Coercible::String
  attribute :age,  Types::Coercible::Int
end

User.new(name: 'Bob', age: '18')
# => #<User name="Bob" age=18>
User.new(name: 'Bob', age: 'not coercible')
# => ArgumentError: invalid value for Integer(): "not coercible"
```

- Use `.optional` to denote that an attribute can be `nil` (see [Optional Values](/gems/dry-types/optional-values)):

```ruby
class User < Dry::Struct
  attribute :name, Types::Strict::String
  attribute :age,  Types::Strict::Int.optional
end

User.new(name: 'Bob', age: nil)
# => #<User name="Bob" age=nil>
# name is not optional:
User.new(name: nil, age: 18)
# => Dry::Struct::Error: [User.new] nil (NilClass) has invalid type for :name
# keys must still be present:
User.new(name: 'Bob')
Dry::Struct::Error: [User.new] :age is missing in Hash input
```

- You can add your own custom constraints (see [Constraints](/gems/dry-types/constraints.html)):

```ruby
class User < Dry::Struct
  attribute :name, Types::Strict::String
  attribute :age,  Types::Strict::Int.constrained(gteq: 18)
end

User.new(name: 'Bob', age: 17)
# => Dry::Struct::Error: [User.new] 17 (Fixnum) has invalid type for :age
```

- Add custom metadata to a type:

```ruby
class User < Dry::Struct
  attribute :name, Types::String
  attribute :age,  Types::Int.meta(info: 'extra info about age')
end
```

... and more.

Note that you don't have to use `Dry::Struct`. You can interact with your
type definitions however you like using `[]`:

```ruby
Types::Strict::String["foo"]
# => "foo"
Types::Strict::String["10000"]
# => "10000"
Types::Coercible::String[10000]
# => "10000"
Types::Strict::String[10000]
# Dry::Types::ConstraintError: 1000 violates constraints
```

### Features

* Support for [constrained types](/gems/dry-types/constraints)
* Support for [optional values](/gems/dry-types/optional-values)
* Support for [default values](/gems/dry-types/default-values)
* Support for [sum types](/gems/dry-types/sum)
* Support for [enums](/gems/dry-types/enum)
* Support for [hash type with type schemas](/gems/dry-types/hash-schemas)
* Support for [array type with members](/gems/dry-types/array-with-member)
* Support for arbitrary meta information
* Support for typed struct objects via [dry-struct](/gems/dry-struct)
* Types are [categorized](/gems/dry-types/built-in-types), which is especially important for optimized and dedicated coercion logic
* Types are composable and reusable objects
* No const-missing magic and complicated const lookups
* Roughly 6-10 x faster than Virtus

### Use cases

`dry-types` is suitable for many use-cases, for example:

  * Value coercions
  * Processing arrays
  * Processing hashes with explicit schemas
  * Defining various domain-specific information shared between multiple parts of your applications
  * Annotating objects

### Other gems using dry-types

`dry-types` is often used as a low-level abstraction. The following gems use it already:

* [dry-struct](/gems/dry-struct)
* [dry-initializer](/gems/dry-initializer)
* [Hanami](http://hanamirb.org)
* [rom-rb](http://rom-rb.org)
* [Trailblazer](http://trailblazer.to)
