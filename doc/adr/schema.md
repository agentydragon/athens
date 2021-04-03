# Schema

TODO: add prior art - RDF, etc.

TODO: would be nice to have a command "pull out anonymous node"

## The case for allowing users to define a schema over their graph

### Graph text editors

Athens (and related software) are currently basically good editors for hypertext
editors. They let you write rich text pages, consisting of blocks, and very
easily link them together and follow those links. These links induce a directed
graph over your, well, Athens graph.

### The graph is good for organizing

One important benefit of having your notes laid like this is that you can use
the graph to organize them. Querying over the graph (beyond the "normal mode" of
seeing blocks in the pages they belong to) provides "alternative views" of the
graph. One way to think of those "alternative views" is as a hierarchy in your
notes that exists parallel to the "pages have blocks, blocks are structured as a
tree" hierarchy.

#### Examples of how page references create useful alternate hierarchies

* Roam, Athens, Logseq all have a "TODO/DONE" tag used to mark blocks with TODO
  tasks, and done tasks. That way, if a user wants to see all their tasks-to-do,
  they can just open the list of references to the `TODO` page.
* Many users have pages for people, and they tag mentions of the person with
  their page. For example:
  * `[[John Doe]] asked me to shop for groceries later this week`
  * `Meeting participants: [[John Doe]], [[Katie Elk]]`
  Looking up the references of the `John Doe` page lets the user see anything
  that involved the person John Doe. For example, if you remember "oh, John
  asked me for something, what was it", you can just look up the "shop for
  groceries later this week" block from John Doe's references.

### Users adopt their own "standards" to keep their notes organized

Currently pages are referenced only with a manual action by the user (i.e.,
`[[...]]`, `#...`, or linking unlinked references). (It would be, however, good
to explore (semi)-automating this in the future.)

That means that if a user wants to get the benefit of structure from references,
they need to consistently tag pages where they should be tagged - like every
time there's a relevant mention of John Doe, tag it with `[[John Doe]]`.

### There are baked in tag systems, but they're not open

Athens, Roam etc. have some support for making some of this tagging easier:

* The `{{[[TODO]]}}` widget has a dedicated keyboard shortcut.
* Roam has a `/date` command for easily referencing daily pages for a given date.
* Athens links YouTube embeds to a `youtube` page.

However, those "out-of-the-box tagging systems" only cover specific tagging
schemes implemented by the products - simple tasks, dates, etc.

Users do not have affordances to help them create their own custom-built
hierarchies. Cooking enthusiasts might want hierarchies of recipes
(appetizer/main/side/salad/..., Italian/Chinese/Mexican/...). Machine learning
engineers might want a hierarchy like "paper/technique/concept/...",
"classification/regression/unsupervised/reinforcement learning/...", "to
read/finished reading/...".

### Keeping the needed "standards" to maintain structure is hard

Currently, Athens and Roam work as text editors. The graph structure is created
from the text you type in. If you type in the wrong text, you will create the
wrong structure.

When you create a new page (or write notes), if you want to keep a useful
structure, you have to remember to add the proper tags to your notes. Users have
invented "headers" that they insert into new page, possibly with help from
text expanders or Roam templates.

Example:

```
* Dish type: [[Appetizer]] / [[Main]] / [[Side]] / [[Salad]] / ...
* Cuisine: [[Italian]] / [[Chinese]] / [[Mexican]] / ...
* Difficulty: [[Easy]] / [[Medium]] / [[Hard]]
* Serves: 4
* Taken from: http://www.example.org/beans-with-awesome-sauce
```

A user might have templates like "recipe", "person", "meeting", etc., each with
cues to fill in some attributes the user wants on such pages.

But even with this workaround, there's still just Markdown-like text underneath.
When you're writing a new dish page, nothing is stopping you from accidentally
writing `[[Mexico]]` instead of `[[Mexican]]`. Then, if you feel like cooking
mexican and open references to the `Mexican` page, you will not find the recipe.

If your friend John Doe also often goes by Johnny, it's easy to accidentally
create a duplicate page `Johnny Doe` when `John Doe` already exists. (TODO:
link synonyms issue)

### Schemas invite better than plain-text UIs

If we make Athens understand the concept of "the user has some standard schema
in their notes", this schema can make Athens usable as a graph database.

This would allow Athens to act as general purpose, but specializable software,
for any domain the user can create a schema for. With some UI on top of it,
Athens could be useful in a similar way Excel is. When you need more structure
than a plain text file, but you don't want to install or build full custom
software.

Some ideas follow.

#### Validation

Let's say that you, as a chef, define your cookbook schema, and tell Athens:

* My nodes are of these types: recipes, dish types, cuisines, difficulty.
* The `Cuisine` attribute of any recipe should point to a cuisine node.

Then, you start a new recipe, and write in `[[Mexico]]` instead of `[[Mexican]]`
into the `Cuisine` attribute, and either linked it to all your trips to Mexico,
or created a new page `Mexico`. Athens looks at the `Mexico` page, and notices
that it is not a cuisine node (e.g., does not contain `* Type: [[Cuisine]]`,
is not on a whitelist, etc.). Athens can then flag this error for you:

```
* Cuisine: [[Mexico]]      <-- Warning: value should be a cuisine node
```

Sometimes, you start out with an unstructured database, and want to impose new
structure on top of it. For example, you've so far collected 10 recipes, and by
the 11th recipe, you decide you'd like to tag them with their cuisines.

Assuming that you already tagged them as recipes, Athens could help by leting
you create a new validation rule ("all recipes should have a cuisine in their
Cuisine attribute"), and let you see recipes which do not yet conform to that.

#### Dropdowns

The natural next step is that when the user creates a recipe page, Athens could
show a dropdown/combobox UI with valid / recommended options:

* Cuisine: [[           v]]
            --------------
	    |    Italian |
	    |    Chinese |
	    |    Mexican |
	    |        ... |

#### Tables

When a chef maintains their cookbook in a Roam graph, it would be natural to
want to view it as a sortable table with the standard attributes:

|-------------------------|----------------|--------------|------------------------------|
| Recipe                  | Dish type      | Cuisine      | Difficulty                   |
|-------------------------|----------------|--------------|------------------------------|
| [[Ramen]]               | [[Soup]]       | [[Japanese]] | [[Medium]]                   |
| [[Spaghetti carbonara]] | [[Main]]       | [[Italian]]  | [[Easy]]                     |
| ...                     | ...            | ...          | ...                          |

Athens right now can search for all mentions of `Recipe`, which could pull up
all blocks tagged with `[[Recipe]]`, and hence also contain the recipe pages (as
breadcrumbs for these blocks), but there's right now no sorting, and especially
no sorting like "sort on the content of the 'Difficulty' block".

If we can show entities with some selected attributes as a table like this,
Athens becomes more usable as a cookbook, but also as, for example, an issue
tracker.

As extras, we could make the tables editable - for example, "Difficulty" column
could be a dropdown offering the valid options.

#### Graphs

If another user uses a hierarchy to organize their tasks, they might want to
visualize dependencies between tasks (which have their own dedicated pages) as
a graph:

```
[[Draft design doc]] → [[Implement prototype]]
				     ↘
                                    [[Collect metrics]] → [[Finish project]]
				     ↗                       ↗
      [[Fix the bug]] → [[Deploy fix]]       [[Minify JavaScript]]
```

Athens can visualize the structure of the graph of *references*, whereas this
would be a graph extracted from attributes like:

```
# Collect metrics

* Depends on: [[Implement prototype]], [[Deploy fix]]
* [[2020-01-01]]
  * Talked with [[John Doe]], he think we can do this before we [[Minify
    JavaScript]].
```

The unstructured block mentioning `Minify JavaScript` *does not* create a
dependency edge between `Collect Metrics` and `Minify JavaScript`.

## Proposed implementation

TODO: Roam Research already has attributes.
Let's start by parsing them, and allowing users to define rules based on them.
