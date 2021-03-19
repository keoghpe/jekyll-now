---
layout: post
title: A Simple Database in Ruby
---

I've been reading Martin Kleppmann's book "Designing Data-Intensive Applications" and decided to create a simple project
to explore and solidify my understanding of some of the concepts explained in his book. The code for this
post can be found [here](https://github.com/keoghpe/rb-db).

At the start of chapter 3, Kleppmann describes a simple key value store that stores data by appending the latest write to
a log file. Kleppmann implements the database as two bash functions, which I've decided to implement with Ruby. 
I'm also taking the opportunity to try out `sorbet` for Ruby type checking.

The set function is fairly simple, it takes an integer key and a hash value. To converts the hash to JSON and writes
appends it to the file. This is an O(1) operation - very efficient.

```ruby
# typed: strict
# frozen_string_literal: true

require_relative "db/version"
require 'sorbet-runtime'
require 'json'

module Rb
  module Db
    class Database
      LOG_FILE = T.let("./aof.txt", String)

      extend T::Sig
      sig {void}
      def initialize
        @log = T.let(File.open(LOG_FILE, 'a+'), File)
        @log.sync = true
      end

      sig {params(key: Integer, value: T::Hash[T.untyped, T.untyped]).returns(NilClass)}
      def set(key, value)
        @log.write("#{key},#{value.to_json}\n"); nil
      end

    end

    class Error < StandardError; end
  end
end

```

We can test out this implementation with IRB:

```irb
2.5.3 :001 > require_relative './lib/rb/db'
 => true
2.5.3 :002 > db = Rb::Db::Database.new
 => #<Rb::Db::Database:0x00007f8db6069550 @log=#<File:./aof.txt>>
2.5.3 :003 > db.set(1, {first_name: 'Peter', last_name: 'Keogh'})
 => nil
2.5.3 :004 > db.set(2, {first_name: 'Steve', last_name: 'Jobs'})
 => nil
2.5.3 :005 > db.set(3, {first_name: 'Steve', last_name: 'Wozniak'})
 => nil
2.5.3 :006 > db.set(1, {first_name: 'Pete', last_name: 'Keogh'})
 => nil
```

And we can see the logfile:

```bash
rb-db ➤ tail -F aof.txt 
1,{"first_name":"Peter","last_name":"Keogh"}
2,{"first_name":"Steve","last_name":"Jobs"}
3,{"first_name":"Steve","last_name":"Wozniak"}
1,{"first_name":"Pete","last_name":"Keogh"}
```

How about retrieval? We can retrieve the value for a given key by iterating over every line in the log file,
matching the lines that contain that key, returning the last value and parsing it's JSON.

```ruby
  sig {params(key: Integer).returns(T::Hash[T.untyped, T.untyped])}
  def get(key)
    File.open(LOG_FILE, 'r')
        .each_line.grep(/^#{key},/).last
        &.gsub(/^\d+,/, '')&.yield_self {|h| JSON.parse(h)}
  end
```

Usage:

```irb
2.5.3 :007 > db.get(1)
 => {"first_name"=>"Pete", "last_name"=>"Keogh"}
2.5.3 :008 > db.set(1, {first_name: 'Steve', last_name: 'Wozniak'})
 => nil
2.5.3 :009 > db.get(1)
 => {"first_name"=>"Steve", "last_name"=>"Wozniak"}
```

This works, but is very inefficient - it's O(n) as we need to iterate over every line to retrieve the latest key. Kleppmann
introduces a hash index to solve this problem.

## Hash Indexes

We can use a hash index to perform reads in O(1) time. The index is an in memory hash that uses the user defined key as
a key and the byte offset in the file as a value.

We update our constructor to initialize the index:

```ruby
  sig {void}
  def initialize
    @log = T.let(File.open(LOG_FILE, 'a+'), File)
    @log.sync = true
    @index = T.let({}, T::Hash[Integer, Integer])
  end
```

When writing, we get the byte offset (with `file.pos`) and store it with the key:
```ruby
  sig {params(key: Integer, value: T::Hash[T.untyped, T.untyped]).returns(NilClass)}
  def set(key, value)
    line = "#{key},#{value.to_json}\n"
    @log.write(line)
    byte_offset = @log.pos - line.length
    @index[key] = byte_offset
    nil
  end
```

When reading, we open the file, seek to that byte offset and read the line:

```ruby
  sig {params(key: Integer).returns(T::Hash[T.untyped, T.untyped])}
  def get(key)
    file = File.open(LOG_FILE, 'r')
    file.pos = T.must(@index[key])
    file.readline.gsub(/^\d+,/, '').yield_self {|h| JSON.parse(h)}
  end
```

The above works fine, but what about when we close our program and reopen it? It turns
out that reads are returning `nil` for indexes we've already added to the database! To prevent
this, we need to populate the index when the DB is first created by calling `populate_index` 
from the constructor.

```ruby
  sig { returns(NilClass) }
  def populate_index
    file = File.open(LOG_FILE, 'r')
    loop do
      begin
        current_pos = file.pos
        line = file.readline
        key = T.must(line.match(/^(\d+),/))[1].to_i
        @index[key] = current_pos
      rescue EOFError
        break
      end
    end
  end
```

To Do:
- Create flash cards for different file modes.
- Explain the different file modes.
- Explain why you open the file multiple times.
- Does the byte offset approach work for UTF-8?
