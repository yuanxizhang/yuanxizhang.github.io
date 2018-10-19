---
layout: post
title:      "Make a CLI Ruby Gem with Bundler"
date:       2018-10-19 05:25:43 +0000
permalink:  make_a_cli_ruby_gem_with_bundler
---


Making and publishing your own gem is simple if you use the tool [Bundler](https://bundler.io).

To install Bundler:  `$ gem install bundler`
To check the version:  `$ bundler -v`
To update Bundler:  `$ gem update bundler` 

We will make a gem called "no_starch_press" that help us find books that are published by [No Starch Press](https://nostarch.com/).

`$ bundler gem no_starch_press`

Step1: Specify your dependencies in no_starch_press.gemspec file:
```
spec.add_development_dependency 'pry'
spec.add_dependency "nokogiri"
```
Inside no_starch_press.rb, add the bundled environment:
```
require 'nokogiri'
require "open-uri"

require_relative "no_starch_press/version"
require_relative 'no_starch_press/CLI'
require_relative 'no_starch_press/topic'
require_relative 'no_starch_press/book'
```
Step 2: Create a CLI user interface in CLI.rb:

```
class NoStarchPress::CLI 
  def call 
    scrape_topic
    list_topics
    menu
    goodbye
  end 
  
  def scrape_topic
    NoStarchPress::Topic.scrape_topics
  end 
  
  def list_topics
    @topics = NoStarchPress::Topic.all
    @topics.each.with_index(1) do |topic, i| 
      puts "Topic #{i}. #{topic.name}"
      
    end
  end
  
  # define a function called menu
	# ask for user input "Enter a number for the topic you want, type list to see all topics or type exit to leave: "
  
  def goodbye
    puts "See you later for another book search!"
  end
end
```

Step 3: Create a topic class to get the topics in topic.rb, each topic has many books, so the realtionship between topic and book is the "has-many" kind. Topic has three attributes: name, url, an array of books.

```
class NoStarchPress::Topic
  
  attr_accessor :name, :url, :books
  
  @@all = []
  
  def initialize(name = nil, url = nil, books = [])
    @name = name
    @url = url
    @books = []
  end
  
  def self.all 
    @@all.uniq 
  end
  
  def self.clear_all 
    @@all.clear
  end 
  
  def self.scrape_topics
    self.clear_all
    doc = Nokogiri::HTML(open("https://nostarch.com/"))
    
    doc.css("div.views-field span.field-content a").each do |element|
      topic = self.new
      topic.name = element.text.strip
      topic.url = "https://nostarch.com#{element.attribute("href").text.strip}"
      topic.books = topic.get_books
      @@all << topic if ( topic.name != "Gift Certificates" and (not @@all.include?(topic)))
    end 
    
    @@all.uniq
  end 
  
  def get_books
    @books.clear
    doc = Nokogiri::HTML(open(@url))
    
    doc.css("div.product-title a").each do |element|
      book = NoStarchPress::Book.new
      book.title = element.text.strip
      book.url = "https://nostarch.com#{element.attribute("href").text.strip}"
      book.topic = self if book.topic == nil
    
      @books << book unless @books.include?(book)
    end
    @books.uniq
  end 
  
  def add_book(book)
    if book.topic == nil
      book.topic = self
    end
    
    unless @books.include?(book)
      @books << book
    end
  end
  
end
```

Step 4: Build a book class in book.rb, the book class has three attributes: title, url, and topic.

```
class NoStarchPress::Book
  attr_accessor :title, :url
  attr_reader :topic
  @@all = []
  
  def initialize(title = nil, url = nil, topic = nil)
    @title = title
    @url = url
    self.topic = topic if topic
    @@all << self unless @@all.include?(self)
  end 
  
  def topic=(topic)
    @topic = topic 
    topic.add_book(self) 
  end
  
end
```
Step 5: Build the Ruby gem:

`$ gem build no_starch_press.gemspec`



