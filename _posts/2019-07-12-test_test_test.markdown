---
layout: post
title:      "The Magic of ActiveRecord"
date:       2019-07-12 07:26:41 -0400
permalink:  test_test_test
---


In the final project for the Sinatra section, I decided to create a [Recipe Manager](https://github.com/tgray017/recipe-manager) app for users to be able to create, view, update and delete recipes. Obviously, one key component to such an app would be the ability to create and edit ingredients. Instead of making this a simple textarea field where a user lists all their ingredients which are then stored within the `recipes` table, I decided to create a separate table, and therefore a separate model for `Ingredient`s. The alternative seemed messy and less flexible; what if I wanted to allow users to see a list of recipes for which chicken was an ingredient? That would require parsing through each recipe's ingredients attribute and selecting any that had the word "chicken" in it, which didn't feel quite right.

Instead, I chose to implement this by creating input fields for the user to enter the ingredient's name (e.g. basil), quantity (e.g. 2), and unit (e.g. tsp) into a table within the `new` and `edit` views of the larger recipe. Originally, this is how the ingredients table within the view looked, with `ingredients` having its own nested hash within the `recipe`'s parent hash:

```
# other input fields
<table id="ingredient_table">
  <th>Ingredient</th>
  <th>Quantity</th>
  <th>Unit</th>
  <tr>
    <td><input type="text" name="recipe[ingredients][][name]"></td>
    <td><input type="text" name="recipe[ingredients][][quantity]"></td>
    <td><input type="text" name="recipe[ingredients][][unit]"></td>
  </tr>
</table>
# other input fields
```

The only issue I came across with this implementation was the controller logic for *updating* an existing recipe. In an ideal world, I would have been able to update a recipe with a one-liner:
```
@recipe.update(params[:recipe])
```
However, ActiveRecord had no way of knowing that the nested `ingredients` hash related to anything other than the `Recipe` model, so I kept getting errors trying to call `#update` on `@recipe`.

After much googling, I discovered some ActiveRecord magic: [#accepts_nested_attributes_for](https://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html#method-i-accepts_nested_attributes_for).
All I would have to do is add `accepts_nested_attributes_for :ingredients` to my Recipe model, so that's what I did! I also needed to make a slight adjustment to my view, which was to replace all instances of the `name="recipe[ingredients][][name]"` input with `name="recipe[ingredients_attributes][][name]"`. The reason being that `accepts_nested_attributes_for :ingredients` actually provides a method called `ingredients_attributes=` which is responsible for updating the ingredients and requires the nested hash to have that same naming convention. The ingredients input table ended up looking like this:

```
# other input fields
<table id="ingredient_table">
  <th>Ingredient</th>
  <th>Quantity</th>
  <th>Unit</th>
  <tr>
    <td><input type="text" name="recipe[ingredients_attributes][][name]"></td>
    <td><input type="text" name="recipe[ingredients_attributes][][quantity]"></td>
    <td><input type="text" name="recipe[ingredients_attributes][][unit]"></td>
  </tr>
</table>
# other input fields
```

I finally got `@recipe.update(params[:recipe])` to work, and now my code is much prettier. Thanks ActiveRecord, I don't know what I'd do without you.


