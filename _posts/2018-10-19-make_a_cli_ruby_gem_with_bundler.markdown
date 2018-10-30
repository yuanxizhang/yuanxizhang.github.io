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
Step 2: Create a CLI user interface in CLI.rb. This is the controller that asks user to select a topic from a given list and shows the books in that topic the user selected.

```
class NoStarchPress::CLI 
  def call 
    scrape_topic
    list_topics
    menu
    goodbye
  end 
  
  def scrape_topic
    NoStarchPress::Scraper.scrape_topics
  end 
  
  def list_topics
    NoStarchPress::Topic.all.each.with_index(1) do |topic, i|
      puts "Topic #{i}. #{topic.name}"
    end
  end
  
  # define a function called menu that interacts with the user
  def menu
    puts "Welcome to No Starch Press - the finest in geek entertaiment!"

    input = nil
    while input != "exit"
      puts "Enter a number for the topic you want, type list to see all topics or type exit to leave: "
      input = gets.strip.downcase
      if input.to_i > 0 and input.to_i <= NoStarchPress::Topic.all.count
        NoStarchPress::Scraper.get_books(NoStarchPress::Topic.all[input.to_i - 1]) if NoStarchPress::Topic.all[input.to_i - 1].books.count == 0

        puts "There are #{NoStarchPress::Topic.all[input.to_i - 1].books.count} "+ NoStarchPress::Topic.all[input.to_i - 1].name + " books: "

        NoStarchPress::Topic.all[input.to_i - 1].books.each.with_index(1) do |book, i|
          puts "Book #{i}. " + book.title
          puts "           " + book.url
        end
      elsif input == "list"
        list_topics
      elsif input == "exit"
        break
      else
        puts "Not sure which topic you want, type list or exit: "
      end
    end
  end	
	
  def goodbye
    puts "See you later for another book search!"
  end
end
```

Step 3: Create a topic class to make new topics in topic.rb, each topic has many books, so the realtionship between topic and book is the "has-many" kind of relationship. The Topic object has three attributes: name, url, and an array of books. The array of books are a collection of instances of the Book class that share the same topic.

```
class NoStarchPress::Topic
  
  attr_accessor :name, :url, :books
  
  @@all = []
  
  def initialize(name = nil, url = nil, books = [])
    @name = name
    @url = url
    @books = []
    @@all << self if ( self.name != "Gift Certificates")
  end
  
  def self.all 
    @@all.uniq 
  end
  
  def self.clear_all 
    @@all.clear
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

Step 4: Build a book class to make new books in book.rb. The book object has three attributes: title, url, and topic. the data type for a book's title and url is string, the book's topic is an instance of the Topic class, the book belongs to that topic.

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

Step 5: Create a Scraper class that scrape the nostarch.com web site for list of topics and books.

```
class NoStarchPress::Scraper
  def self.scrape_topics

    doc = Nokogiri::HTML(open("https://nostarch.com/"))

    doc.css("div.views-field span.field-content a").each do |element|
      topic = NoStarchPress::Topic.new
      topic.name = element.text.strip
      topic.url = "https://nostarch.com#{element.attribute("href").text.strip}"
    end
  end

  def self.get_books(topic)

    doc = Nokogiri::HTML(open(topic.url))

    doc.css("div.product-title a").each do |element|
      book = NoStarchPress::Book.new
      book.title = element.text.strip
      book.url = "https://nostarch.com#{element.attribute("href").text.strip}"
      book.topic = topic
      topic.books << book unless topic.books.include?(book)
    end
  end
end
```

Step 6: Build the Ruby gem:

`$ gem build no_starch_press.gemspec`



