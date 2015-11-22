# Misc. Thoughts on Ruby

## Accessibility API Mapping

- [Requirements](html-aam/requirements.md)

## Auto hiding ruby annotation

### Focus region of ruby annotations

The fallback contents of a canvas element can be associated with the regions of screen.

Is it possible to associate hidden ruby text with ruby base?

How do browsers handle `<ruby>foo bar baz<rt>foo <a href="#">bar</a> baz</ruby>`?

Should hidden ruby annotation be displayed if the element or one of its descendants gain the focus?

```css
rt:has(:focus) { visibility: visible; }
```

### CSS Ruby

- [Proposal to make "hidden" ruby annotation not have an associated CSS layout box](css-ruby/hidden-ruby-annotation.html)

### innerText

```js
const root = document.getElementById('test');
root.innerHTML = '<ruby lang="ja"><rb>とある<rb>科学<rb>の<rb>超電磁砲</rb><rp>（</rp><rt>とある<rt>かがく<rt>の<rt>レールガン</rt><rp>）</rp></ruby>';
console.log(root.innerText); // -> 'とある科学の超電磁砲（とあるかがくのレールガン）';
```

### contentEditable

Should hidden ruby annotation be displayed if the element or its parent ruby element is editable?
