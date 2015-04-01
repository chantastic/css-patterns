# CSS Patterns

This is how I'm writing CSS. It's hard to say if these ideas are objectively
"good." I can say that when I write this way things seem to fall into place and
I paint myself into fewer corners.

# The 140 character pitch

Ultimately scalable CSS without fake-modules or silly naming. It's modeled after
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

If you have a `.veggie-burger`, it should decorate the more generic `.burger`
class. How do we do that without extension?

```html
<div class="veggie-burger burger">...</div>
<div class="bean-burger   burger">...</div>
```

## Language Parts

### Nouns

Lots of nouns. Let in rain.

```css
.person { ... }
.select { ... }
.cat { ... }
.burger { ... }
.btn { ... }
.rainbow { ... }
```

### Attribute Noun

Use an underscore for class attributes: `.noun_attribute`

Let's say a `.cat` has a `.breed`. `.breed` could also be  a first-class noun of
it's own. In a Rails app, Breed might be a normalized resource.

In this case `.cat` **belongs_to** a `.breed`, I'll use a single underscore.

`.cat_breed`

This suggests that `.cat_breed` isn't a decorator on `.breed` but an attribute
of `.cat`.

This frees us up to use `.cat-breed` as a `.breed` decorator.

### Adjectives

Adjectives decorate a noun. Do the same with your classes; separate with a dash.

```css
.irritating-person { ... }
.large-select { ... }
.spayed-cat { ... }
.veggie-burger { ... }
.dangerous-btn { ... }
.double-rainbow { ... }
```

### verbs

Verbs should be reserved for actions. These would both be legal if used on page
actions:

```css
.destroy-btn { ... }
.show-btn { ... }
```

_used very similar to adjectives_

### state-verbs (is/has/can)

State should be represented by a class with one verb prefix.

Verb-prefixed classes must never have a definition in the global scope:

```css
.is-current { /* illegal */ }
.person.is-current { /* legal */ }
```

I use `is-` as my prefix. I try to stick exclusively to `is`. If I myself doing
verbal gymnastics just use us `is`, I'll try `has` (or `can` as a very last
resort).

## SOLID

This approach follows interpretations of SOLID. Here's how.

### Single responsibility

`.burger` should do only one thing&mdash;visually represent a `.burger`. If it's
expected to do more, it should be decorated to do that.

```css
.burger {
  display: block;
  color: pinkish
}

.veggie-burger { color: green }

.bean-burger { color: lightbrown }
```

```html
<div class="burger">  ... </div>
<div class="veggie-burger burger"> ... </div>
<div class="bean-burger   burger"> ... </div>
```

It is not appropriate is to change `.burger` based on context:

```css
/* illegal */
.veggie .burger { ... }
.bean .burger { ... }
```

### open-closed

Eagerly decorate classes; resist changing them.

```css
.person { ... }
.admin-person { ... }
.owner-admin-person { ... }
```

```html
<div class="person"> ... </div>
<div class="admin-person person"> ... </div>
<div class="owner-admin-person person"> ... </div>
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
decorate. `.admin-person` depends on `.person`. `owner-admin-person` depends on
both `admin-person` and `person`.

`.person` can be replaced substitute class that fulfills the same expectations:

```css
.person       { display: inline-block }

.admin-person { background-color: gold }

.mock-inline  { display: inline-block }
```

```html
<!-- all the same -->
<div class="admin-person person"> ... </div>
<div class="admin-person mock-inline"> ... </div>
<div class="admin-person" style="display: inline-block"> ... </div>
```

# Further Reading

* [ROCSS, for working with JSON APIs](https://github.com/chantastic/rocss)

# Objections

## What about the idea of not coupling styles to models

I see this as a Scale&reg; concern. Most systems I've worked are worse for
being prematurely concerned with "scale" and normalizing classes to early.
