# Patterns for CSS Modules & Atomic Classes

### Explore this guide
- [Beware the "class list order fallacy"](#beware-the-class-list-order-fallacy)
- [Creating overridable default styles](#creating-overridable-default-styles)

---

<details>
  <summary>Learn more: <strong>Terminology</strong></summary>
  <hr>
  <blockquote>
    <ul>
      <li>
        <strong><em>Atomic classes:</em></strong> Also known as "utility classes" or "helper classes."
      </li>
      <li>
        <strong><em>CSS Modules:</em></strong> An approach to pseudo-scoping classes and other custom identifiers in CSS (e.g. animation names, named grid lines, and named grid template areas), by prefixing these names with a hash, preventing name collisions. This transformation happens during the build process.
      </li>
    </ul>
  </blockquote>
</details>

---

<br>

### Beware the "class list order fallacy"
The order of classes defined in a `className` attribute does not affect how the rules may override each other. It is the order in which those classes _are loaded_ into the CSS engine which matters.

Consider the following JSX element which we'll suppose is from a reusable component. It uses a class from a CSS Module and a class list coming from our component props. We want the developers consuming our reusable component (let's call them our "consumers") to be able to override any of the styles we've applied.

```jsx
<div className={`btn btn-secondary ${styles.niceDefaults} ${classNames}`}>
```

We might have written the above assuming that `classNames` should always over `.btn`, `.btn-primary`, and `.niceDefaults`, due to the order of this class list. But we're actually relying on the classes supplied by `classNames` to be loaded into the browser _before_ the others.

Consider: What if `classNames` contains `.btn-primary`? That's a Bootstrap class which is defined in the `bootstrap.css`/`bootstrap.min.css` _before_ `.btn-secondary`. It is also certainly loaded into browser _before_ our component's custom CSS Module. Therefore our default style `.btn-secondary` will override the consumer's class, and so will our `.niceDefaults`.

That's unintended behavior. We don't want our consumers to need `!important`, waste valuable time debugging their CSS, or resorting to CSS Modules or Inline Styles unnecessarily. As much as possible, their solution should "just work." So what do we do?

<sup>\[[Jump to the top](#patterns-for-css-modules--atomic-classes)\]</sup>

<br><br>

### Creating overridable default styles

---

<details>
  <summary>Learn more: <strong><em>The Technical Challenge</em></strong></summary>
  <hr>
  <blockquote>
    <ol>
      <li>
        Incoming classes from props, Bootstrap classes, and our own component's custom classes will all usually match a selector with a specificity value of 10, and generally _should._ When two conflicting rules match in terms of specificity, <strong>the last to be read by the CSS engine will win</strong>.
      </li>
      <li>
        Bootstrap classes are defined in <strong>an order which is hidden away</strong> from us in an inconvenient-to-reference bundle file. Sometimes the order can be guessed, but this is unreliable.
      </li>
      <li>
        As for incoming classes from props and our own component's custom classes, it is, for a variety of reasons, <strong>impossible to be certain</strong> which class will be read by the CSS engine first except through experimentation.
      </li>
      <li>
        Bootstrap utility classes commonly use `!important`, <strong>elevating the rules beyond specificity</strong> (this can be thought of as ∞ specificity). That's normally fine for content development (which shouldn't need to be overridden), but it's bad news for a style rule specifically intended to be overridden.
      </li>
      <li>
        Together these traits make it easy to design UI components which resist later styling and confound expectations, contributing to the common complaints that CSS development is fraught with time-consuming trial-and-error and mysterious behavior.
      </li>
    </ol>
    <p><em>Therefore...</em></p>
  </blockquote>
</details>

---

1. **Identify the styles which ought to be overridable.** In terms of [Atomic Design](https://www.justinmind.com/ui-design/atomic-design), these are usually "Atoms" and "Molecules" – reusable UI components. These components usually _receive_ user-facing content as `props` or `children` exclusively. Content comes from the outside.
2. **Avoid Bootstrap for styles which ought to be overridable.** Though _generally,_ we want to use Bootstrap for most daily development for our web app (that is, content development), this is an exception.
3. **Use the [`:where(.foo)` selector](https://developer.mozilla.org/en-US/docs/Web/CSS/:where) in a CSS Module to define overridable styles.** This pseudo-class reduces the specificity of the provided selector to 0, making these styles easy to override.

```css
/* These styles and those from any nested style blocks will depend
   on the :where() selector, which will present no problem to anyone
  seeking to provide their own styles. */
:where(ul.defaultStyles) {
  list-style: none;
  padding: 0;

  & > li {
    display: inline;
    margin-right: 1rem;
  }
}
```

<sup>\[[Jump to the top](#patterns-for-css-modules--atomic-classes)\]</sup>

<br><br><br><br>
