# Requirements for reading ruby annotations

## Principal: supporting reading preferences

- [Issue 427756 - chromium - Accessibility text-to-speech doesn't respect `<ruby>` element](https://code.google.com/p/chromium/issues/detail?id=427756)

Users should able to choose whther screen readers read ruby bases or ruby annotations (ruby texts).

- Reading ruby without rtc elements
- Reading ruby with rtc elements

## Reading ruby without rtc elements

```html
<ruby><rb>base<rt>annotation</ruby>
```

Assistive technologies may need to do following steps to read ruby bases and not to read ruby annotations:

- base
- annotation

- For each child of ruby elements:
	- if the child is either a phrasing content node or a rb element: process the child
	- otherwise: skip the child

### Complexity 1:

The process can be complicated as ruby base and ruby texts can be intermixed. W3C HTML has following example:

```html
<ruby lang="ja"><rb>日<rt>に</rt><rb>本<rt>ほん</rt><rb>語<rt>ご</rt></ruby>
```

- 日本語
- にほんご

```html
<ruby lang="ja">常用<rp>(</rp><rt>じょうよう<rp>)</rp>漢字<rp>(</rp><rt>かんじ<rp>)</rp>表<rp>(<rp><rt>ひょう</rt><rp>)</rp></ruby>
```

- 常用漢字表
- じょうようかんじひょう

DOM tree (and Accessibility Tree) of this will look like:

- ruby
	- rb
		- \#text (base)
	- rt
		- \#text (annotation)
	- rb
		- \#text (base)
	- rt
		- \#text (annotation)
	- rb
		- \#text (base)
	- rt
		- \#text (annotation)

This can't be fixed.

### Complexity 2:

The process can be more complicated as phrasing content nodes are allowed as ruby children and they treated as if ruby element is ommitted. Let's consider following example:

```html
<ruby>東<rb>京<rp>(<rt>とう<rt>きょう<rp>)</ruby>
```

DOM tree (and Accessibility Tree) will look like:

- ruby
	- \#text (base)
	- rb
		- \#text (base)
	- rt
		- \#text (annotation)
	- rt
		- \#text (annotation)

```html
<ruby lang="ja"><time datetime="05-05">端午</time><mark>節句</mark><rp>（<rt>たんごのせっく<rp>）</ruby>
```

Note: "節句" means "seasonal celebration"

DOM tree (and Accessibility Tree) will look like:

- ruby
	- time
		- \#text (base)
	- mark
		- \#text (base)
	- rp
		- \#text
	- rt
		- \#text (annotation)
	- rp
		- \#text

Assistive technologies need to process any contents other than rp or rt elements.

### Simplification 2


Above process can be simplified if rb element is implicitly constructed like tbody element.

```html
<ruby>東<rb>京<rp>(<rt>とう<rt>きょう<rp>)</ruby>
```

```html
<ruby><rb>東<rb>京<rp>(<rt>とう<rt>きょう<rp>)</ruby>
```

DOM tree (and Accessibility Tree) will look like:

- ruby
	- rb
		- \#text (base)
	- rb
		- \#text (base)
	- rp
		- \#text
	- rt
		- \#text (annotation)
	- rt
		- \#text (annotation)
	- rp
		- \#text


```html
<ruby lang="ja"><time datetime="05-05">端午</time><mark>節句</mark><rp>（<rt>たんごのせっく<rp>）</ruby>
```


```html
<ruby lang="ja"><rb><time datetime="05-05">端午</time><mark>節句</mark><rp>（<rt>たんごのせっく<rp>）</ruby>
```

DOM tree (and Accessibility Tree) will look like:

- ruby
	- rb
		- time
			- \#text (base)
		- mark
			- \#text (base)
	- rp
		- \#text
	- rt
		- \#text (annotation)
	- rp
		- \#text

Genralized form is followings

- ruby
	- one or more rb elements
	- zero or more rp elements
	- one or more rt elements
	- zero or more rp elements
	- one or more rb elements
	- zero or more rp elements
	- one or more rt elements
	- zero or more rp elements

The required process of assistive technologies will be simplified as follows:

To read the ruby bases:

- For each child of ruby elements:
	- if the child is a rb element: process the child
	- otherwise: skip the child

To read the ruby annnotations:

- For each child of ruby elements:
	- if the child is rt element: process the child
	- otherwise: skip the child


## Reading ruby with rtc elements

TODO: Is rtc element really needed? It seems too complicated.

Using rtc element, authors can annotate a base twice.

For example:

```html
<ruby lang="ja"><rb>東南</rb><rp>（</rp><rtc><rt>とうなん</rt></rtc><rp>、</rp><rtc><rt>たつみ</rt></rtc><rp>）</rp></ruby>
```

- ruby
	- rb
	- rtc
		- rt
	- rtc
		- rt

- ruby base is "東南"
- 1st ruby annotation is "とうなん"
- 2nd ruby annotation is "たつみ"

Reading both 1st and 2nd annotations usually doesn't make sense. Japanese speakers usually don't pronounce "東南" as "とうなんたつみ". They usually choose either "とうなん" or "たつみ".

Users may want to know all of the ruby annotations especially in the education. In that cases users shoud know boundary of each ruby anotations cluster. In other words, users should know the position or index of ruby annotations cluster.

- For each child of ruby elements:
	- if the child is contained by n-th (anonymous) ruby text container: process the child
	- otherwise: skip the child

However determining whether an element is contained by n-th (anonymous) ruby text container is complicated.

### Complexity 3

The rtc element can be ommited entirely.


```html
<ruby lang="ja"><rb>東南</rb><rp>（</rp><rt>とうなん</rt><rp>、</rp><rtc><rt>たつみ</rt></rtc><rp>）</rp></ruby>
```

- ruby
	- rb
	- rt
	- rtc
		- rt

How to divid ruby annnotations to ruby annotation container is described in W3C HTML.

Assistive Technologies need to implement that algorithm.

The algorithm is complicated.

For example, each (anonymous) ruby text container can have different number of rt elements.

```html
<ruby><rb>旧<rb>金<rb>山<rt>jiù<rt>jīn<rt>shān<rtc>San Francisco</ruby>
```

- ruby
	- rb
	- rb
	- rb
	- rt
	- rt
	- rt
	- rtc

Assistive Technologies treat that as if

- ruby
	- rb
	- rb
	- rb
	- rtc
		- rt
		- rt
		- rt
	- rtc


- 旧金山
- jiùjīnshān
- San Francisco


Additionaly, one ruby element can have multiple pairs of ruby bases and ruby containers

```html
<ruby>
  ♥<rp>: </rp><rt>Heart<rp>, </rp><rtc lang=fr>Cœur<rp>.</rp>
  ☘<rp>: </rp><rt>Shamrock<rp>, </rp><rtc lang=fr>Trèfle<rp>.</rp>
  ✶<rp>: </rp><rt>Star<rp>, </rp><rtc lang=fr>Étoile<rp>.</rp>
</ruby>
```

- ruby
	- \#text
	- rt
	- rtc
	- \#text
	- rt
	- rtc
	- \#text
	- rt
	- rtc

Assistive Technologies treat that as if

- ruby
	- rb
	- rtc
		- rt
	- rb
	- rtc
		- rt
	- rb
	- rtc
		- rt


- ♥ ☘ ✶
- Hear Shamrock Start
- Cœur Trèfle Étoile


### Simplification 2

Above process can be simplified if a rtc element are implicitly constructed like tbody element.

```html
<ruby>
  ♥<rp>: </rp><rt>Heart<rp>, </rp><rtc lang=fr>Cœur<rp>.</rp>
  ☘<rp>: </rp><rt>Shamrock<rp>, </rp><rtc lang=fr>Trèfle<rp>.</rp>
  ✶<rp>: </rp><rt>Star<rp>, </rp><rtc lang=fr>Étoile<rp>.</rp>
</ruby>
```

- ruby
	- rb
	- rtc
		- rt
	- rtc

	- rb
	- rtc
		- rt
	- rtc

	- rb
	- rtc
		- rt
	- rtc


```html
<ruby><rb>旧<rb>金<rb>山<rt>jiù<rt>jīn<rt>shān<rtc>San Francisco</ruby>
```

```html
<ruby><rb>旧<rb>金<rb>山<rtc><rt>jiù<rt>jīn<rt>shān</rtc><rtc><rt>San Francisco</rt></rtc></ruby>
```

- ruby
	- rb
	- rb
	- rb
	- rtc
		- rt
		- rt
		- rt
	- rtc

Generalized form is:

- ruby
	- one or more rb element(s)
	- rtc
		- one or more phrasing content nodes or rt elements
	- rtc
		- one or more phrasing content nodes or rt elements
	- one or more rb element(s)
	- rtc
		- one or more phrasing content nodes or rt elements
	- rtc
		- one or more phrasing content nodes or rt elements

To read ruby bases:

- For each child of ruby elements:
	- if the child is a rb element: process the child
	- otherwise: skip the child

To read n-th ruby annotation:

- let index is 0
- For each child of ruby elements:
	- if the child is a rb element: let index is 0
	- if the child is a rtc element:
		- incrment index
		- if index equals to n: process the child
		- otherwise: skip the child
	- otherwise: skip the child
