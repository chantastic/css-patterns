# CSS Patterns

This is how I'm writing CSS. It's hard to say if these ideas are "good." I can
say that when I write this way things seem to fall into place and I paint myself
into fewer corners.

# The 140 character pitch

Scalable CSS without fake-modules or silly naming. It's modeled after
SOLID and Ruby decorators.

## The decorator pattern

Decorators are nice because they are scalable ad infinitum.

When naming a decorator, you typically add specifiers to the left. Here's an
example:

```ruby
Person
AdminPerson

Burger
VeggieBurger
```

If this fictional example were in Ruby, `VeggieBurger` might decorate `Burger`
and be instantiated with a new `Burger` when created. `VeggieBurger` doesn't
`extend` burger. It wraps it with additional behavior or attributes.

This is what we're trying to mimic in CSS.

If you have a `.VeggieBurger`, it should decorate the more generic `.Burger`
class. How do we do that without extension?

```html
<div class="VeggieBurger Burger">...</div>
<div class="BeanBurger   Burger">...</div>
```

## Language Parts

### Nouns

Lots of nouns. Let in rain.

```css
.Person { ... }
.Select { ... }
.Cat { ... }
.Burger { ... }
.Btn { ... }
.Rainbow { ... }
```

### Attribute Noun

Use an underscore for class attributes: `.Noune_attribute`

Let's say a `.Cat` has a `.Breed`. `.Breed` could also be  a first-class noun of
it's own. In a Rails app, Breed might be a normalized resource.

In this case `.Cat` **belongs_to** a `.Breed`, I'll use a single underscore.

`.Cat_breed`

This suggests that `.Cat_breed` isn't a decorator on `.Breed` but an attribute
of `.Cat`.

This frees us up to use `.CatBreed` as a `.Breed` decorator.

### Adjectives

Adjectives decorate a noun. Do the same with your classes; separate with a dash.

```css
.IrritatingPerson { ... }
.LargeSelect { ... }
.SpayedCat { ... }
.VeggieBurger { ... }
.DangerousBtn { ... }
.DoubleRainbow { ... }
```

### verbs

Verbs should be reserved for actions. These would both be legal if used on page
actions:

```css
.DestroyBtn { ... }
.ShowBtn { ... }
```

_used very similar to adjectives_

### state-verbs (is/has/can)

State should be represented by a class with one verb prefix.

Verb-prefixed classes must never have a definition in the global scope:

```css
.is-current { /* illegal */ }
.Person.is-current { /* legal */ }
```

I use `is-` as my prefix. I try to stick exclusively to `is`. If I myself doing
verbal gymnastics just use us `is`, I'll try `has` (or `can` as a very last
resort).


## File system

This approach wins on the file system.

Because classes are well named, each class should have it's own file:

```
.
..
Select.css
PersonSelect.css
ShowPersonSelect.css
LargeShowPersonSelect.css
```

Contrast the way other conventions are typically written to files:

```
.
..
Select.css
```

In that file would be all of the "modifiers" associated with that file.

Yes, there's nothing keeping you from writing out files for modifiers. But this
seems nonsensical:

```
.
..
Select.css
Select--small.css
Select--large.css
```

The thinking is out of place because select in bound to need more
context-specific modifications. There are too many questions for answers. Does
it justify a new file? Do I tack on context to an existing
modifier(`Select--small--person-show`)? Is this modification reusable? Should it
be(`Select--small--not-as-small-as-small-though`).

The decorator pattern answers those questions.

* New File? Yes
* Do a add context to an existing class? Yes
* Is the new class reusable in unrelated contexts? No

## Composition

Classes are composed of a single class which may be composed of a single class,
which may be composed of a single class, at infinitum. This is *not* inheritance
or extension. A decorator has a dependency on all of its more generic classes.

```
+---------------------------+
| LargeShowPersonSelect.css |
|                           |
| +-------------------------+
| |    ShowPersonSelect.css |
| |                         |
| |   +---------------------+
| |   |    PersonSelect.css |
| |   |                     |
| |   |     +---------------+
| |   |     |    Select.css |
+-+---+-----+---------------+
```

## Errors

It's possible in a system like this to have errors.

```css
.LargeShowPersonSelect:not(.ShowPersonSelect),
.LargeShowPersonSelect:not(.PersonSelect),
.LargeShowPersonSelect:not(.PersonSelect) {
  position: relative !importante;
}

.LargeShowPersonSelect:not(.ShowPersonSelect)::before,
.LargeShowPersonSelect:not(.PersonSelect)::before,
.LargeShowPersonSelect:not(.PersonSelect)::before {
  position: absolute !important;
    width: 100% !important;
    height: 100% !important;
  content: "CSS error" !important;
  background-color: red !important;
  color: white !important;
}

.LargeShowPersonSelect:not(.ShowPersonSelect) {
  content: "dependency .show-person-select not provided";
}

.LargeShowPersonSelect:not(.PersonSelect) {
  content: "dependency .person-select not provided";
}

.LargeShowPersonSelect:not(.PersonSelect) {
  content: "dependency .select not provided";
}
```

## SOLID

This approach follows interpretations of SOLID. Here's how.

### Single responsibility

`.Burger` should do only one thing&mdash;visually represent a `.Burger`. If it's
expected to do more, it should be decorated to do that.

```css
.Burger {
  display: block;
  color: pinkish
}

.VeggieBurger { color: green }

.BeanBurger { color: lightbrown }
```

```html
<div class="Burger">  ... </div>
<div class="VeggieBurger Burger"> ... </div>
<div class="BeanBurger   Burger"> ... </div>
```

It is not appropriate is to change `.burger` based on context:

```css
/* illegal */
.Veggie .Burger { ... }
.Bean .Burger { ... }
```

### open-closed

Eagerly decorate classes; resist changing them.

```css
.Person { ... }
.AdminPerson { ... }
.OwnerAdminPerson { ... }
```

```html
<div class="Person"> ... </div>
<div class="AdminPerson Person"> ... </div>
<div class="OwnerAdminPerson Person"> ... </div>
```

A simple rule is this: the fewer nouns/adjectives/etc., the more resistant you should
be to changing that class. It must apply to *ALL* downstream classes.

### Liskov Substitution Principle / Interface Segregation Principle

Composition > Inheritance.

`@extend` is the kind of tricky. You'll be tempted to use it. Resist that
temptation. Your code will be better, more adaptable, and easier to reason
about if you resist.

Err on the side of creating too many classes.

### Dependency Inversion Principle

Classes that decorate have an implicit dependency on the classes that they
decorate. `.AdminPerson` depends on `.Person`. `.OwnerAdminPerson` depends on
both `.AdminPerson` and `.Person`.

`.Person` can be replaced substitute class that fulfills the same expectations:

```css
.Person       { display: inline-block }

.AdminPerson { background-color: gold }

.mock-inline  { display: inline-block }
```

```html
<!-- all the same -->
<div class="AdminPerson Person"> ... </div>
<div class="AdminPerson mock-inline"> ... </div>
<div class="AdminPerson" style="display: inline-block"> ... </div>
```

# Further Reading

* [ROCSS, for working with JSON APIs](https://github.com/chantastic/rocss)

# Objections

## What about the idea of not coupling styles to models

I see this as a Scale&reg; concern. Most systems I've worked are worse for
being prematurely concerned with "scale" and normalizing classes to early.
