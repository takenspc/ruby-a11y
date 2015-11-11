# Misc. Thoughts on Ruby

## CSS Ruby

- [Proposal to clarify "hidden" ruby annotation does not have an associated CSS layout box](css-ruby/hidden-ruby-annotation.html)

```
<ruby>
	<rb>とある</rb><rb>科学</rb><rb>の</rb><rb>超電磁砲</rb>
	<rp>（</rp>
	<rt>とある</rt><rt>かがく</rt><rt>の</rt><rt>レールガン</rt>
	<rp>）</rp>
</ruby>
```

```
<ruby>
	<rb>とある</rb><rb>科学</rb><rb>の</rb><rb>超電磁砲</rb>
	<rp>（</rp>
	<rt><a href="?1">とある</a></rt><rt><a href="?2">かがく</a></rt><rt><a href="?3">の</a></rt><rt><a href="?4">レールガン</a></rt>
	<rp>）</rp>
</ruby>
```
