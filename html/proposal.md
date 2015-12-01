# Proposals for the W3C HTML ruby model

```html
<ruby><rb>base<rt>annotation</ruby>
```

## Summary

W3C HTML has "a [ruby model] that matches both users' needs and implementations". However the current ruby model is not easy to implement. Specifically, the assistive technologies need to implement all of the ruby model because browsers don't expose the ruby structure to ATs in the current ruby model. This proposal aims to make the ruby model easier to implement by assistive technologies.

This proposal intends to

- preseve the makrup of the current ruby model
- change start and end tag of [`rb`] and [`rtc`] elements be omissible like [`tbody`]

In other words, this proposal intends to make the every DOM child of `ruby` element be either `rb`, `rtc` or `rp` element.

   [ruby model]: http://www.w3.org/TR/html51/semantics.html#the-ruby-element
   [`rb`]: http://www.w3.org/TR/html51/semantics.html#the-rb-element
   [`rtc`]: http://www.w3.org/TR/html51/semantics.html#the-rtc-element
   [`tbody`]: http://www.w3.org/TR/html51/semantics.html#the-tbody-element

## Requirements: supporting reading preferences

[Issue 427756 - chromium - Accessibility text-to-speech doesn't respect `<ruby>` element](https://code.google.com/p/chromium/issues/detail?id=427756)

A user should be able to choose whether a screen reader read ruby bases or ruby annotations (ruby texts).

To do that, screen readers need to distingush between the ruby bases and ruby annotations.

This proposal discuss single-sided ruby model (ruby without `rtc` elements) followed by discussions of the double-sided ruby (ruby texts with `rtc` elements). Because double-sided ruby is much more complicated.

## Single-sided ruby (ruby without `rtc` elements)

```html
<ruby><rb>base<rt>annotation</ruby>
```

Assistive technologies may need to do following steps to distingush between ruby bases and ruby annotations of single-sided ruby:

- let `last is rb` be false
- let `n` be 0
- For each `child` of a `ruby` element:
	- if the `child` is <mark>either a phrasing content node or a `rb` element</mark>:
		- let `last is rb` is false
			- increment `n`
			- let `last is rb` is true
		- the child is a part of ruby base of `n`-th ruby segment
	- if the child is a `rt` element:
		- let `last is rb` be false
		- the child is a part of ruby annotation of `n`-th ruby segment

### Complexity 1: Implicit ruby bases

The process can be complicated as phrasing content nodes are allowed as children of `ruby` element and they treated as ruby bases as if `rb` element is omitted. Let's consider following example:

Example 1

```html
<ruby>東<rb>京<rp>(<rt>とう<rt>きょう<rp>)</ruby>
```

Note: The ruby base of Example 1 is "東京" and ruby annotation is "とうきょう".

The DOM tree (and Accessibility Tree) of Examle 1 will look like:

- ruby
	- \#text (東)
	- rb
		- \#text (京)
	- rt
		- \#text (とう)
	- rt
		- \#text (きょう)

This means that screen readers need to handle first text node as a part of ruby base as described in [W3C HTML ruby model] while browsers already handled the ruby element for rendering.

   [W3C HTML ruby model]: http://www.w3.org/TR/html51/semantics.html#segmentation-and-categorisation-of-ruby

Consider more complicated example:

Example 2

```html
<ruby><time datetime="05-05">端午</time><mark>節句</mark><rp>（<rt>たんごのせっく<rp>）</ruby>
```

Note: The ruby base of Example 2 is "端午節句" and the ruby annotation is "たんごのせっく".

Note: "節句" means "seasonal celebration"

DOM tree (and Accessibility Tree) of Example 2 will look like:

- ruby
	- time
		- \#text (端午)
	- mark
		- \#text (節句)
	- rp
		- \#text
	- rt
		- \#text (たんごのせっく)
	- rp
		- \#text

Assistive technologies need to process any contents other than `rp` or `rt` elements.

Note: According to the current HTML AAM specification, [`time`] and [`mark`] elements construct accessible objects.

   [`time`]: TODO
   [`mark`]: TODO

### Proposal 1: make start and end tag of `rb` be omissible

Above complexity can be reduced if `rb` element is implicitly constructed like `tbody` element. In other words, if start and end tag of `rb` can be omitted.

Example 1:

```html
<ruby>東<rb>京<rp>(<rt>とう<rt>きょう<rp>)</ruby>
```

will be treated as

Example 1a:

```html
<ruby><rb>東<rb>京<rp>(<rt>とう<rt>きょう<rp>)</ruby>
```

DOM tree (and Accessibility Tree) of Example 1 and Example 1a will look like:

- ruby
	- rb
		- \#text (東)
	- rb
		- \#text (京)
	- rp
		- \#text
	- rt
		- \#text (とう)
	- rt
		- \#text (きょう)
	- rp
		- \#text

Example 2:

```html
<ruby lang="ja"><time datetime="05-05">端午</time><mark>節句</mark><rp>（<rt>たんごのせっく<rp>）</ruby>
```

will be treated as

Example 2a:

```html
<ruby lang="ja"><rb><time datetime="05-05">端午</time><mark>節句</mark><rp>（<rt>たんごのせっく<rp>）</ruby>
```

DOM tree (and Accessibility Tree) of Example 2 and Example 2a will look like:

- ruby
	- rb
		- time
			- \#text (端午)
		- mark
			- \#text (節句)
	- rp
		- \#text
	- rt
		- \#text (たんごのせっく)
	- rp
		- \#text

The required process of assistive technologies will be simplified as follows:

- let `last is rb` be false
- let `n` be 0
- For each `child` of a `ruby` element:
	- if the `child` is <mark>a `rb` element</mark>:
		- let `last is rb` is false
			- increment `n`
			- let `last is rb` is true
		- the child is a part of ruby base of `n`-th ruby segment
	- if the child is a `rt` element:
		- let `last is rb` be false
		- the child is a part of ruby annotation of `n`-th ruby segment


## Double-sided ruby (ruby with `rtc` elements)

TODO: Is `rtc` element really needed? It seems too complicated.

Using `rtc` element, authors can annotate a base twice. For example:

Example 3:

```html
<ruby><rb>東南<rp>（</rp><rtc><rt>とうなん</rt></rtc><rp>、</rp><rtc><rt>たつみ</rt></rtc><rp>）</rp></ruby>
```

In this example, the ruby base is "東南", the first ruby annotation is "とうなん" and the second ruby annotation is "たつみ".

Reading both first and second annotations usually doesn't make sense. Japanese speakers usually don't pronounce "東南" as "とうなんたつみ". They usually pronounce either "とうなん" or "たつみ".

Thus that screen readers need to distingush each ruby annnotation container. To do that screen readers need to determine:

- how many ruby text annoations are there
- which ruby annotation segment is belong to which ruby text container

So that assistive technologies may need to do following steps:

- let `last is rb` be false
- let `n` be 0
- let `m` be 0
- For each child of a `ruby` element:
	- if the child is a `rb` element:
		- if `last is rb` is false
			- increment `n`
			- let `last is rb` be true
		- the child is a part of ruby base of `n`-th ruby segment
	- if the child is is <mark>contained by n-th (anonymous) ruby text container</mark>:
		- if `last is rb` is true:
			- let `m` be 0
			- let `last is rb` be false
		- increment `m`
		- the child is `m`-th ruby annotation contaienr of `n`-th ruby segment

I know the sentence "the child is is contained by n-th (anonymous) ruby text container" is tautological. However determining whether an element is contained by n-th (anonymous) ruby text container is actually complicated.

### Complexity 2: Implicit ruby text containers

In the current W3C HTML ruby model, the `rtc` element can be entirely omited.

Example 3 can be written as follows:

Example 3a

```html
<ruby><rb>東南</rb><rp>（</rp><rt>とうなん</rt><rp>、</rp><rtc><rt>たつみ</rt></rtc><rp>）</rp></ruby>
```

The DOM tree (and Accessibility Tree) of Examle 3a will look like:

- ruby
	- rb
	- rt
	- rtc
		- rt

How to divide ruby annotations to ruby annotation container is described in W3C HTML. And assistive Technologies need to implement that algorithm while browers already do that algorithm for rendering.

Even worse, the algorithm can be more complicated than Example 3a. For example, each (anonymous) ruby text container can have different number of rt elements.

Example 4

```html
<ruby><rb>旧<rb>金<rb>山<rt>jiù<rt>jīn<rt>shān<rtc>San Francisco</ruby>
```

Note: The ruby base is "旧金山", the first ruby annotation is "jiùjīnshān", the second ruby annotaion is "San Francisco".

The DOM tree (and Accessibility Tree) of Example 4 will look like:

- ruby
	- rb (旧)
	- rb (金)
	- rb (山)
	- rt (jiù)
	- rt (jīn)
	- rt (shān)
	- rtc (San Francisco)

The assistive technologies need to treat Example 4 as if

- ruby
	- rb (旧)
	- rb (金)
	- rb (山)
	- rtc
		- rt (jiù)
		- rt (jīn)
		- rt (shān)
	- rtc
		- rt (San Francisco)

Additionaly, one `ruby` element can have multiple pairs of ruby bases and ruby containers

Example 5

```html
<ruby>
  ♥<rp>: </rp><rt>Heart<rp>, </rp><rtc lang=fr>Cœur<rp>.</rp>
  ☘<rp>: </rp><rt>Shamrock<rp>, </rp><rtc lang=fr>Trèfle<rp>.</rp>
  ✶<rp>: </rp><rt>Star<rp>, </rp><rtc lang=fr>Étoile<rp>.</rp>
</ruby>
```

In this example, there are 3 ruby segements.

- first segment: the ruby base is "♥", the first ruby annnotation is "Heart", and the second ruby annotation is "Cœur"
- second segment: the ruby base is "☘", the first ruby annnotation is "Shamrock", and the second ruby annotation is "Trèfle"
- third segment: the ruby base is "✶", the first ruby annnotation is "Star", and the second ruby annotation is "Étoile"

The DOM tree (and Accessibility Tree) of Example 5 will look like:

- ruby
	- \#text (♥)
	- rt (Heart)
	- rtc (Cœur)
	- \#text (☘)
	- rt (Shamrock)
	- rtc (Trèfle)
	- \#text (✶)
	- rt (Star)
	- rtc (Étoile)

The assistive technologies need to treat that as if

- ruby
	- rb (♥)
	- rtc
		- rt (Heart)
	- rtc
		- rt (Cœur)
	- rb (☘)
	- rtc
		- rt (Shamrock)
	- rtc
		- rt (Trèfle)
	- rb (✶)
	- rtc
		- rt (Star)
	- rtc
		- rt (Étoile)

This is too much requirements for assistive technologies.


### Proposal 2: make start and end tag of `rtc` be omissible

Above complexity can be reduced if `rtc` element is implicitly constructed like `tbody` element. In other words, if start and end tag of `rtc` can be omitted.

If so Example 4

```html
<ruby>
  ♥<rp>: </rp><rt>Heart<rp>, </rp><rtc lang=fr>Cœur<rp>.</rp>
  ☘<rp>: </rp><rt>Shamrock<rp>, </rp><rtc lang=fr>Trèfle<rp>.</rp>
  ✶<rp>: </rp><rt>Star<rp>, </rp><rtc lang=fr>Étoile<rp>.</rp>
</ruby>
```

constructs the DOM tree (and Accessibility Tree) look like:

- ruby
	- rb (♥)
	- rtc
		- rt
	- rtc
		- rt
	- rb (☘)
	- rtc
		- rt
	- rtc
		- rt
	- rb
	- rtc
		- rt
	- rtc
		- rt

This is much cleaner tree.

So that assistive technologies may need to do following steps:

- let `last is rb` be false
- let `n` be 0
- let `m` be 0
- For each child of a `ruby` element:
	- if the child is a `rb` element:
		- if `last is rb` is false
			- increment `n`
			- let `last is rb` be true
		- the child is a part of ruby base of `n`-th ruby segment
	- if the child is <mark>a `rtc` element</mark>:
		- if `last is rb` is true:
			- let `m` be 0
			- let `last is rb` be false
		- increment `m`
		- the child is `m`-th ruby annotation contaienr of `n`-th ruby segment
