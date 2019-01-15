---
layout: post
title:      "Combining Meta-programming with Mass Assignment"
date:       2019-01-15 09:36:31 -0500
permalink:  combining_meta-programming_with_mass_assignment
---


In one of the lessons in the Flatiron curriculum, we learned a little bit about mass assignment--that is, assigning instance variables dynamically based on data instead of explicitly defining each one individually. For instance (pun intended), say we have a `Character` class, and each instance of a character contains a hash called `properties`, with different attributes associated with each property based on the character. 

For my final project, I scraped a bunch of different wiki pages, each of which housed its own `Character` from the TV show "Game of Thrones", and created a `properties` hash for each character based on the information in that wiki page. This hash would include key-value pairs like `:name => ["Eddard Stark"]`, `:spouse => ["Catelyn Stark"]`, and `:title => ["Lord of Winterfell", "Warden of the North", "Hand of the King"]`. However, I wanted to be a good little programmer and use good OO ruby conventions, so instead of keeping this properties hash as-is I decided to turn each key into setter/getter methods whith the goal of returning the associated value when calling the getter method.

To assign the values using the setter methods, I could create an instance method like this, which we learned about in the Flatiron curriculum:
```
def assign_character_properties(properties)
	properties.each {|k, v| self.send("#{k}=", v)}
end 
```
This, however, would require explicitly defining `attr_accessors` for each potental key in the ruby file (otherwise I'd get a `NoMethod` error). But because my project involved scraping many different wiki pages, that would mean defining many different `attr_accessors`, because although some of these keys overlapped across all characters (e.g. `:house`, `:family`, `:gender`), others were more obscure and were only used by a small subset of characters (e.g. `:significant_other`, `:voiced_by`, :`motion_picture`). Because combing through each individual wiki page to determine all the potential `attr_accessors` felt wrong, and because I'm lazy, I turned to Google for a metaprogramming solution. 

It turns out that the solution is way simpler than I thought it would be--that same `#send` method can be used for `attr_accessors`, but instead of operating on the instance of the class like in the previous example, it would need to operate on the class itself. This turned out to be quite elegant, if I do say so myself:
```
def assign_character_properties(properties)
    properties.each do |k, v|
      self.class.send(:attr_accessor, k) unless self.class.instance_methods.include?(k)
      self.send("#{k}=", v)
    end
end
```
Since I'm calling the `#each` method on the `properties` hash, which is assigned for each _instance_ of the `Character` class, I'm being sure to grab that instance's class (`Character`), and `#send` setter and getter methods for whatever property is being iterated on, unless the setter and getter methods already exist from a previous character's iteration over their `properties` hash. Then, once the methods have been created, I assign the associated value of the property to the instance variable for which the `attr_accessor` was just created. From an iterative approach, it looks like this:

Iteration 1:
```
# Class 
class Character
	attr_accessor :name
end

# Instance
=> #<Character:0x000000031a2310
 @name=["Eddard \"Ned\" Stark"]>
```

Iteration 2:
```
#Class
class Character
	attr_accessor :name, :spouse
end

#Instance
=> #<Character:0x000000031a2310
 @name=["Eddard \"Ned\" Stark"],
 @spouse=["Catelyn Stark"]>
```
Iteration 3:
```
#Class
class Character
	attr accessor :name, :spouse, :title
end

#Instance
=> #<Character:0x000000031a2310
 @name=["Eddard \"Ned\" Stark"],
 @spouse=["Catelyn Stark"],
 @title=>["Lord of Winterfell", "Warden of the North", "Hand of the King"]>
```
Of course, you can't actually see those `attr_accessors` in the ruby file, but that's effectively what's happening. 

Now that our instance variables are set for each instance of a `Character`, how do we dynamically call each of the getter methods? If we want to take everything we just assigned and output it to `$stdout`, we can use the following (I have this defined in my `cli.rb` file since it's dealing with output, which is why I'm explicitly calling the `Character` class instead of `self`):

```
def display_character_properties(char_name)
    Character.all.each do |char|
      if char.name == char_name
        char.class.instance_methods(false).each do |meth|
          unless meth.to_s[-1] == "=" || char.send("#{meth}").nil?
            puts "#{meth.to_s.gsub("_", " ").capitalize}:"
            char.send("#{meth}").each {|val| puts "   #{val}"}
          end
        end
      end
    end
  end 
```
Here we're iterating over all the characters (`#all` is a pre-defined class variable that contains all instances of the `Character` class), grabbing the character whose name matches the name that was passed in to `display_character_properties`, grabbing all of the getter methods that were defined from our `properties` hash for that character as long as the return value isn't `nil`, and outputting a human-friendly name of the instance method along with its return value. Continuing with the Ned Stark example, the output in the console would look like this:
```
Name:
   Eddard Stark
Spouse:
   Catelyn Stark
Title:
   Lord of Winterfell
   Warden of the North
   Hand of the King
```

This approach is extremely useful when dealing with a lot of data where you don't need to implement custom constructors, and you want to create instance variables out of whatever data is thrown at you. This is why I believe that meta-programming and mass assignment can be awesome tools for any ruby programer!

*See my full CLI repo here: [https://github.com/tgray017/got_character_query](https://github.com/tgray017/got_character_query). *
