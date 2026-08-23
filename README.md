# talk-quarto-editable

Slides for a talk about editable Quarto documents. Built with Quarto revealjs and animated with [anime.js](https://animejs.com) v3.

## Layout

| Path | What it is |
| --- | --- |
| `index.qmd` | The slides. |
| `all-the-js-code.html` | All animation code, injected via `include-after-body`. |
| `styles.scss` | Theme, animated-slide layout, cursor and drag-in styles. |
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

### Notes

- Write the attributes without a `data-` prefix, as above. Quarto rewrites unknown div attributes on the way out, so `from="left"` becomes `data-from="left"` in the rendered HTML; the code reads either spelling.
- Cursors are absolutely positioned inside the `<section>`, so a slide hosting `.drag-in` content gets `position: relative` and `overflow: visible` automatically. Content parked off-slide is clipped by `.reveal` itself.
- Reveal's default fragment transition is `all`, which fights anime.js over the `transform`. `styles.scss` narrows it to `opacity` for `.drag-in` and lets anime own the movement.
- `transform` does not apply to non-replaced inline elements, so `span.drag-in` gets `display: inline-block` in `styles.scss`. Without it a `[text]{.drag-in}` span would sit there and refuse to move. Images are fine either way.
- On the way out, Reveal drops the `visible` class immediately, so the code adds `.drag-leaving` to hold the element on screen until the animation finishes.
- Positions are computed at the moment the fragment fires, from the element's resting layout position. Nothing needs hardcoded coordinates, and the effect survives window resizes between fragments.

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
