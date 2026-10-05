
${family.header()}

# [[Meal Plan|Dinner this week]]
${family.meals()}

# Shopping list
* [ ] Milk
* [ ] Mushrooms
* [ ] Risotto rice
* [ ] Mozzarella
* [ ] Apples
* [ ] Toothpaste
* [x] Bread
* [ ] Birthday candles

# Chores
${family.chores()}

# [[Calendar|Coming up]]
${family.events()}

# How this is built
Plain markdown pages, plus a few SilverBullet features:
* [Tasks](https://docs.silverbullet.md/Task): the shopping list is a checklist right on this page. Chores are tasks in [[Chores]] that name who does them, like `@finn`.
* [Attributes](https://docs.silverbullet.md/Attribute): meals and events are list items with a day or date attached, in [[Meal Plan]] and [[Calendar]].
* [Queries](https://docs.silverbullet.md/Space%20Lua/Integrated%20Query): dinner this week, chores per person and what’s coming up are read from those pages.
* [Space Lua](https://docs.silverbullet.md/Space%20Lua) and [Space Style](https://docs.silverbullet.md/Space%20Style): the fridge door and the other widgets live in [[Library/Family Home]].
