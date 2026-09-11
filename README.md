# talk-quarto-editable

Slides for a talk about editable Quarto documents. Built with Quarto revealjs and animated with [anime.js](https://animejs.com) v3.

## Layout

| Path | What it is |
| --- | --- |
| `index.qmd` | The slides. |
| `all-the-js-code.html` | The float and collaborative-editing slides, plus the four original fragment styles (`.drag-in`, `.type-in`, `.multi-cursor`, `.find-replace`) that predate `SlideFx`. |
| `styles.scss` | Theme, animated-slide layout, cursor, and fragment chrome styles. |
| `_extensions/EmilHvitfeldt/cursor-fragments/` | The [quarto-revealjs-cursor-fragments](https://github.com/EmilHvitfeldt/quarto-revealjs-cursor-fragments) extension: `SlideFx`, the registry-based fragment framework, and its library of styles. See that repo's README (and its `gallery.qmd`) for the full style catalogue. |
| `_extensions/revealjs-anime/` | Small plugin that binds animations to RevealJS slide and fragment lifecycle events. Bring your own animation library. |
| `_collab-slide.qmd` | Markup for the collaborative-editing slide. |

Render with `quarto render index.qmd`, or `quarto preview index.qmd` while working.

## Drag-in fragments

A fragment marked `.drag-in` is hauled onto the slide by a little collaborator cursor when it is shown, and hauled back off the same edge when it is hidden (going backwards through the deck).

```markdown
::: {.fragment .drag-in from="left" cursor="Kari"}
Content that a cursor hauls in from the left.
:::
```

All attributes are optional:

| Attribute | Default | Meaning |
| --- | --- | --- |
| `from` | nearest side edge | Edge to enter from: `left`, `right`, `top`, or `bottom`. |
| `cursor` | no label | Name shown on the cursor's tag. Omit for a bare arrow. |
| `color` | next palette color | Cursor color, any CSS color. |
| `duration` | `900` | Length of the haul in milliseconds. |

Several in a row, each with its own collaborator:

```markdown
::: {.fragment .drag-in from="left" cursor="Kari"}
First point.
:::

::: {.fragment .drag-in from="right" cursor="John"}
Second point.
:::

::: {.fragment .drag-in from="bottom" cursor="Hailey" duration="1200"}
Third point, arriving slower.
:::
```

The marker also works on elements *inside* a fragment, which is handy when several things should ride in together on one keypress:

```markdown
::: fragment
![](lizard.gif){.drag-in from="top" width="200"}

[Two things, one keypress.]{.drag-in from="bottom" cursor="Kari"}
:::
```

Each inner element gets its own cursor and its own `from` edge, so they can converge from different directions on a single keypress.

### One cursor drops it wrong, another fixes it

Add `fix="Name"` and the haul becomes a two-cursor bit: the first cursor drops the content slightly off its mark and leaves, then a second cursor flies in from the other side of the slide, grabs it, and nudges it into place.

```markdown
::: {.fragment .drag-in from="left" cursor="Kari" fix="John"}
Kari drops this a bit short, and John nudges it into place.
:::
```

| Attribute | Default | Meaning |
| --- | --- | --- |
| `fix` | off | Name of the correcting cursor. Its presence is what enables the whole bit. |
| `fix-color` | next palette color | Color of the correcting cursor. |
| `fix-delay` | `450` | Beat between the sloppy drop and the correction arriving, in ms. |
| `off` | short of the mark, plus a little cross-axis drift | How wrong the first drop is, as `"x,y"` in slide px relative to the resting position. |

The default miss stops short along the axis the content travelled and drifts a little off it, which is what a real fumbled drag looks like. Override it with `off` for a specific gag, e.g. `off="0,-90"` to drop something too high.

The whole sequence runs about 3.2s at default timings. The correction is built into the same anime.js timeline as the haul rather than scheduled on a timer, so navigating away mid-bit cancels it cleanly, and if you interrupt while the content is still sitting off its mark it gets dragged out from where it actually is rather than snapping to rest first.

### Notes

- Write the attributes without a `data-` prefix, as above. Quarto rewrites unknown div attributes on the way out, so `from="left"` becomes `data-from="left"` in the rendered HTML; the code reads either spelling.
- Cursors are absolutely positioned inside the `<section>`, and a slide hosting `.drag-in` content gets `overflow: visible` so nothing is clipped while off-slide. Content parked off-slide is clipped by `.reveal` itself. Do not add `position: relative` to those sections: Reveal positions sections absolutely, and overriding it drops them into normal flow so every later slide renders in the wrong place. Reveal's own positioning already makes the section a containing block.
- Reveal's default fragment transition is `all`, which fights anime.js over the `transform`. `styles.scss` narrows it to `opacity` for `.drag-in` and lets anime own the movement.
- `transform` does not apply to non-replaced inline elements, so `span.drag-in` gets `display: inline-block` in `styles.scss`. Without it a `[text]{.drag-in}` span would sit there and refuse to move. Images are fine either way.
- On the way out, Reveal drops the `visible` class immediately, so the code adds `.drag-leaving` to hold the element on screen until the animation finishes.
- Positions are computed at the moment the fragment fires, from the element's resting layout position. Nothing needs hardcoded coordinates, and the effect survives window resizes between fragments.

## Type-in fragments

A fragment marked `.type-in` has its text typed out: a cursor flies in, types the content one character at a time while the caret blinks, and leaves.

```markdown
::: {.fragment .type-in cursor="Hailey"}
Hailey types this out, then leaves.
:::

::: {.fragment .type-in cursor="John" from="right" speed="35"}
John is a faster typist.
:::
```

| Attribute | Default | Meaning |
| --- | --- | --- |
| `cursor` | no label | Name shown on the cursor's tag. |
| `color` | next palette color | Cursor color. |
| `speed` | `55` | Milliseconds per character. Lower is faster. |
| `from` | `left` | Edge the cursor arrives from: `left`, `right`, `top`, or `bottom`. |

The cursor tracks the caret as the line grows, so it stays at the point of typing rather than sitting still.

### Notes

- Plain text only. The element's original markup is cached and restored when the typing finishes, so inline formatting survives, but it is not itself typed. Anything with emphasis or links inside will type as flat text and then snap to formatted at the end.
- The block's height is pinned with `min-height` while typing, so a multi-line paragraph does not push the rest of the slide around as it fills in. The lock is released at the end.
- Leading and trailing whitespace is trimmed before typing. A markdown div's `textContent` is padded with newlines, and typing those spends visible keystrokes on invisible characters.
- Going backwards restores the full text immediately and parks the cursor, so the fragment is intact if you return to it, and it retypes from scratch on the way forward again.

## Multi-cursor fragments

A fragment marked `.multi-cursor` has every one of its lines typed at once, each by its own named cursor, all driven from a single tween so the keystrokes land in lockstep. The thing no WYSIWYG tool can do.

```markdown
::: {.fragment .multi-cursor cursors="Kari,John,Hailey,Dave,Iris"}
- setosa
- versicolor
- virginica
- a fourth line
- and a fifth
:::
```

| Attribute | Default | Meaning |
| --- | --- | --- |
| `cursors` | no labels | Comma-separated names, cycled if there are fewer names than lines. |
| `speed` | `55` | Milliseconds per character. |
| `from` | `left` | Edge the cursors arrive from. |

The cursors arrive staggered by 70ms each, which reads better than five appearing at once, but the typing itself is strictly simultaneous. A line shorter than the longest retires early: its caret disappears and its cursor leaves rather than hovering at a finished line.

### Which elements get typed

In order of preference:

1. Any descendants marked `.mc-line`, if there are any. Use this when the automatic choice is wrong.
2. Otherwise the children of the innermost single wrapper. A markdown div holding one `<ul>` resolves to the `<li>` elements rather than to the list as a whole.
3. Otherwise the container itself.

The resolved set is cached on the container. It has to be: the choice depends on which elements have text, and typing empties them, so recomputing on the way out would resolve to the wrong elements and restore nothing.

## Find-and-replace fragments

A fragment marked `.find-replace` gets searched in front of the audience: a find bar drops in, a cursor types the term, every match on the slide highlights at once, the replacement is typed, and Replace All swaps them all simultaneously. The gesture is familiar from every editor; hitting every match at once is the part a WYSIWYG tool cannot do.

```markdown
::: {.fragment .find-replace find="lizard" replace="gecko" cursor="Kari" bar="bottom"}
The lizard sits on a rock. A second lizard joins the first lizard, and a
third lizard watches. Every lizard here is a lizard.
:::
```

| Attribute | Default | Meaning |
| --- | --- | --- |
| `find` | required | Term to search for. Matching is case-insensitive, and the original casing of each match is preserved until it is replaced. |
| `replace` | required | Replacement text. |
| `cursor` | no label | Name on the cursor's tag. |
| `color` | next palette color | Cursor color. |
| `speed` | `55` | Milliseconds per character in the bar's fields. |
| `bar` | `top` | Where the find bar sits: `top` or `bottom`. |

The sequence runs about 5.5 seconds: bar in, cursor to the find field, type the term, highlight all matches with a 22ms stagger, type the replacement, press the button, swap every match, fade the highlights, bar and cursor out.

### Notes

- **Use `bar="bottom"` when text reaches the top-right corner.** The bar floats over the content, exactly as a real find bar does, which means it can cover a match. `top` is the default because it is what editors do, but for a slide the bottom-right corner is usually the empty one.
- Matches are found by walking text nodes and wrapping each hit in its own span, so markup already inside the container survives the search.
- The swap is driven off a single counter rather than one timer per match, which keeps it cancellable and makes the sweep deterministic.
- The whole container's markup is cached and restored on the way out, so navigating back leaves the original text with no leftover highlight spans, and the bit can be replayed.
- The find bar sits at `z-index: 860`, below the cursors at `900`. Otherwise the cursor vanishes behind the bar at the exact moment it clicks Replace All.

## The rest of the fragment library

The other twenty styles used in `index.qmd` (`.retype`, `.nudge`, `.paste-in`, `.move-item`, and so on) are built on the same `SlideFx` registry, but they now live in the [quarto-revealjs-cursor-fragments](https://github.com/EmilHvitfeldt/quarto-revealjs-cursor-fragments) extension rather than in this repo. See that repo's README for the full attribute reference and its `gallery.qmd` for a demo slide of each style, and see its source for notes on writing a new style.

## Hand-written animated slides

For anything more involved than a drag-in, a slide gets a marker class and its animation is registered against the lifecycle:

```js
RevealAnime.defineSlide('collab-slide', {
  enter: (section) => { /* ... */ return stopFn; },
  leave: (section) => { /* ... */ },
  fragments: {
    'frag-id': (section) => { /* ... */ return stopFn; },
    'other-id': { show: (section) => {}, hide: (section) => {} },
  },
});
```

A runner returns a `stopFn`, called automatically on slide leave, on fragment hide, and if the same fragment is shown again. `RevealAnime.slideCoordsOf(section, el)` converts an element's screen rectangle into slide-internal coordinates, which is what anime.js `translateX`/`translateY` values on slide children are measured in. The slide is 1280x720 (`RevealAnime.SLIDE_W` / `SLIDE_H`).

The float slide (`.float-slide`) and the collaborative-editing slide (`.collab-slide`) are the two worked examples in `all-the-js-code.html`.
