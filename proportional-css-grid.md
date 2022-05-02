# Clever title here

## Goal
- A grid layout that
  - is always proportional to the viewport's width
  - maintains square cells
  - sets proportional typography that conforms to the layout grid AND ensures that site visitors can still use text sizing commands to maintain maximum accessibility

I can hear your nose wrinkle and your teeth clench.

But that's not hard enough. Let's turn it up to 11
- No JavaScript
- Scrollbars interact with layout *correctly*
  - vertical scrollbars appear when the layout is longer than the viewport
  - horizontal scroll bars are NOT triggered when vertical scrollbars are added to the viewport.

We all know this is must be impossible. After all, searching online for CSS solutions looking for proportional width and height grids[[LINK]]; looking for advice that addresses (or purports to address) unwanted scrollbars[[LIN]]; trying to understand how the `vw` unit as implemented[[LINK]] –and even defined[[LINK]] in standards in the first place– is fundamentally broken; well, it's all pretty discouraging.

I believe I have found a way to achieve the goal. And I think it could be useful.

We're going to deploy some basic modern CSS using `grid`, `calc()`, and `var()` and a secret weapon that enjoys broad browser support even though it's relatively new.

## Solution step by step

Here's a picture of what we're going to build

[[THREE IMAGES of the layout at different sizes]]


### Step 0 – Write the HTML
The HTML is simple (if repetitive; we're going to want a bunch of tiles to experiment with later). 8 rows of `<div class="tile">`s in 12 columns, that will be placed according to the rules of the parent `<div class="layout">`'s grid and then fomatted by some class names and matching selectors after we get the basics accomplished.

``` html
<!DOCTYPE html>
<html lang="en">
    <head>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta charset="utf-8" />
    <title>Tiles</title>
</head>
<body>
    <div class="layout">
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">rect</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div> 
        <div class="tile">tile</div> 
    </div>
</body>
```

### Step 1 – Define some custom properties in CSS

Our layout is going to be based on the *width* of the viewport (it can also be based it in the *height* of the viewport if that makes more sense to your particular situation). Our example layout is going to be 12 columns wide and have infinite number of rows. Each row will be as high as each column is wide.

You never know what the future holds so were going to define these custom properties to be available to any element in a site, including SVG elements, and even XML (no I can't imagine a situation where I'd be placing XML inside of HTML, but the world is a pretty big place, so never say never).

Set the number of columns for the layout. This will get used when we set the base unit measure.

``` css
:root {
  --number-of-columns: 12;
```

Set the "base" measure for the layout. Divide 100% of the layout by the number of columns.

``` css
  --base-unit: calc(100% / var(--number-of-columns));
```

Set the *divisor* for calculating gap measures in the layout. *This is **NOT** the gap size itself.* Instead, this sets the way layout compensation works when the grid is calculated to include gaps. Don't worry we'll get to that soon enough.

``` css
  --gap-divisor: 10;
```

Now we'll go ahead and set the base measure of the gap for the grid layout. Divide the base unit by the gap divisor

``` css
  --gap-unit: calc(var(--base-unit) / var(--gap-divisor));
```

So now in `:root` (and therefore available to all the child elements in our page), we have

``` css
:root {
  --number-of-columns: 12;
  --base-unit: calc(100% / var(--number-of-columns));
  --gap-divisor: 10;
  --gap-unit: calc(var(--base-unit) / var(--gap-divisor));
}
```

We're using `percent` units to calculate the `--base-unit` which will result in a percent value being used by the browser in its layout routine.

The `--gap-unit` is simply 1 tenth the width of the `--base-unit` for our layout.

Defining `--number-of-columns` and `--gap-divisor` separately means you could simply change these values for different layouts.
    
***Note*** *Go ahead and add an* `html{ background-color: lemonchiffon; }` *or similar to visualize the root html element as we go*

### Step 2 – Define the width of `<body>`

We're going for an edge-to-edge layout in our example. This also demonstrates how scrollbars do the *right thing* using this approach.

``` css
body {
  margin: 0;
  background-color: grey;
  width: calc(var(--base-unit) * 12);
}
```

With the `<body>` width defined, we can go onto Step 3.

### Step 3 – Define the `.layout` context

Here's where most of the action happens. First we'll set the background colour for ease of visualization and then set the display to `grid`.

``` css
  .layout {
    background-color: grey;
    display: grid;
```

Next, we start getting into *brass tacks*. Set the layout to have 12 columns *and subtract the width of a gap unit from that value*

Do the same for the rows. `grid-auto-rows` because we want the rows to be infinitely repeatable and have the same value as the column width.

And we want a consistently sized gap that's applied both verically and horizontally so set `gap` property too

``` css
  grid-template-columns: repeat(12, calc(var(--base-unit) - var(--gap-unit)));
  grid-auto-rows: calc(var(--base-unit) - var(--gap-unit));
  gap: var(--gap-unit);
```

If we stopped right now, our naive assumption would be that this should give us a square cell in the grid, right? Sadly it doesn't. Instead it's *infuriatingly close*.

The squares exceed the `.layout` width and trigger unwanted scrollbars increasing our sadness and decreasing our quality of life.

[[IMAGE OF WHAT IS HAPPENING]]

What we're seeing is some kind of weird math result of rounding. Or something. We need a way to compensate for it.

I subjected myself to a few hours of trial and error, because I don't want you to suffer. We need to apply some compenstation for the rounding anomaly. So instead of 

```css
  gap: var(--gap-unit);
```

we multiply it by a factor of correction

```css
  gap: calc(var(--gap-unit) * 1.09);
```

> This factor depends on the number of columns and the size of the gap unit. Change the `--number-of-columns` value, or the `--gap-divisor` value, or **both**, this factor will need to change.

> I found success when I kept the factor as multiplying by 1.nn

> **I THINK I KNOW WHY THIS WORKS.** If a grid gap is 1/10 of the width of a base unit and there are 12 columns, we will get a twelfth columnar gap at the end of the layout. So we need to multiply the gap to remove this. The obvious answer is multiply the `--grid-gap` by 1.1, but that triggers the horizontal scroll bar in Firefox and Chrome on Windows. So 1.09 it is. The edge-to-edge layout is maintained and no scrollbars are triggered.

At this point, our confidence should be pretty high. We've handled the measures smartly and we know we're going to succeed, but the fact of the matter is a little different (Im sure you figured this out a long time ago…)

[[IMAGE OF LAYOUT WITHOUT ASPECT RATIO]]

The height of the rows is calculated using the `--base-unit` which means we're getting `percent` units from the calculations, ***not*** `px` (pixel) units. The row height is being set at a `percent` of the `<html>` element height.

We can't just make the `grid-auto-row` calculation use some unitless measure because we derive it from the `--base-unit` measure which is calculated in `percent`. We need to do this so we can avoid the deadly `vw` *not considering scrollbars* problem. We can't convert the unit either, because adding or subtracting a pixel – which would convert the unit from `percent` to `px` – will wreck the computed values for element widths and we'll fail to reach our goal.

So I hit the books and found a property that we can use to solve our problem.

It has great browser support

[[IMAGE OF CANIUSE TABLE]]

It makes sense when you read it in CSS, and it helps us with proportional layouts of many different kinds, not just squares.

Enter: `aspect-ratio`.

MDN says
> The `aspect-ratio` CSS property sets a preferred aspect ratio for the box, which will be used in the calculation of auto sizes and some other layout functions.

*YES*

We want the preferred aspect ratio to be *square*. ***We want 1:1***.

[[IMAGE OF ALL THE THINGS 1:1]]

Let's add it to the `.layout` and see if it works.

``` css
.layout {
  /* For visualization */
  background-color: grey;

  display: grid;
  grid-template-columns: repeat(12, calc(var(--base-unit) - var(--gap-unit)));
  grid-auto-rows: calc(var(--base-unit) - var(--gap-unit));
  gap: calc(var(--gap-unit) * 1.09 );
  aspect-ratio: 1 / 1;
}
```

and load it in a browser

[[IMAGE OF SQUARE TILE LAYOUT]]

Really? My trust has been abused enough. I'm going to zoom in an take a screen grab of this

[[IMAGE OF A SQUARE WITH MEASURES APPLIED]]

Does it work in Chrome?

[[Image in Chrome]]

Firefox?

[[Image in Firefox]]

On Windows?

[[Image Windows FF and Chrome]]

in my phone?

[[Image iOS]]

my tablet?

[[Image iPadOS]]

[[Image Android]]

I'm STOKED.

It's probably too much to ask of `aspect-ratio`, but can we drop the size assignment on `grid-auto-rows`? And just let `aspect-ratio` work its magic?

``` css
.layout {
  /* For visualization */
  background-color: grey;

  display: grid;
  grid-template-columns: repeat(12, calc(var(--base-unit) - var(--gap-unit)));
  /* grid-auto-rows: calc(var(--base-unit) - var(--gap-unit)); */
  gap: calc(var(--gap-unit) * 1.09 );
  aspect-ratio: 1 / 1;
}
```

[[Image no value on grid auto row]]

Yeah, okay. I'll calm down. `grid-auto-rows` needs to have a value for assigned for `aspect-ratio` to take effect. Let's put it back

``` css
.layout {
  /* For visualization */
  background-color: grey;

  display: grid;
  grid-template-columns: repeat(12, calc(var(--base-unit) - var(--gap-unit)));
  grid-auto-rows: calc(var(--base-unit) - var(--gap-unit));
  gap: calc(var(--gap-unit) * 1.09 );
  aspect-ratio: 1 / 1;
}
```

### Step 4 – Setting type

The last item in the goal is proportional typography that can still be adjusted by the user using browser controls (such as ⌘+ and ⌘-).

In the `:root` selector where we made our custom properties, I tried assigning `--base-unit` to `--font-unit` so when handling typographic stuff I would be thinking about fonts.

``` css
:root {
  --number-of-columns: 12;
  --base-unit: calc(100% / var(--number-of-columns));
  --gap-divisor: 10;
  --gap-unit: calc(var(--base-unit) / var(--gap-divisor));
  --font-unit: var(--base-unit);
}
```

and assigning it to an `<h1>`

``` css
h1 {
  font-size: var(--font-unit);
  margin: 0;
}
```

This did something, but not what I expected.

[[Image of tiny text]]

So I tried multiplying everything a lot (like `example-of-alternative`) and adding values (like `example-of-adding-px`; which also converts the measures but robs us of user zoom controls) but to no avail.

In desperation I decided to try using `vw` to set the typography measure. 

My reasoning was that the layout won't ever exceed the width of the viewport because we're using `percent`, so we'll never have scrollbars. Also if the `vw` unit was scoped to text, and not layout, we'd never encounter issues with `vw`'s problematic definition or implementation.

So now the `:root` rule looks like this

``` css
:root {
  --number-of-columns: 12;
  --base-unit: calc(100% / var(--number-of-columns));
  --gap-divisor: 10;
  --gap-unit: calc(var(--base-unit) / var(--gap-divisor));
  --font-unit: calc(100vw / 12);
}
```

and the browser displays

[[Image of H1 having been adjusted]]

*and I could use keyboard shortcuts to adjust text size too!*

### Step 5 – Additional Details

Now I'm just going to add a few more details to suggest some layout options like some classes and rules for making tiles of different sizes.

In the HTML inside of `<body>`

``` html
<div class="layout">
        <div class="tile">tile</div>
        <div class="tile tile-1-1">tile</div>
        <div class="tile tile-2-1">tile</div>
        <div class="tile tile-3-1">tile</div>
        <div class="tile tile-4-1">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile tile-1-2">rect</div>
        <div class="tile tile-2-2">tile</div>
        <div class="tile tile-3-2"><h1>tile</h1></div>
        <div class="tile tile-4-2"><h2>tile</h2></div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile tile-1-3">tile</div>
        <div class="tile tile-2-3"><h3>tile</h3></div>
        <div class="tile tile-3-3"><h4>tile</h4></div>
        <div class="tile tile-4-3"><h5>tile</h5></div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div>
        <div class="tile">tile</div> 
        <div class="tile tile-2-6">rect</div> 
    </div>
```

and then in CSS

``` css
.tile {
  background-color: tan;
}

/* 1 Unit High */
.tile-1-1 {
  background-color: lightpink;
}
.tile-2-1 {
  grid-column: span 2;
  /* grid-row: span 1; */
  background-color: lightpink;
}
.tile-3-1 {
  grid-column: span 3;
  /* grid-row: span 1; */
  background-color: lightpink;
}
.tile-4-1 {
  grid-column: span 4;
  /* grid-row: span 1; */
  background-color: lightpink;
}

/* 2 Units High */
.tile-1-2 {
  grid-column: span 1;
  grid-row: span 2;
  background-color: lightgreen;
}
.tile-2-2 {
  grid-column: span 2;
  grid-row: span 2;
  background-color: lightgreen;
}
.tile-3-2 {
  grid-column: span 3;
  grid-row: span 2;
  background-color: lightgreen;
}
.tile-4-2 {
  grid-column: span 4;
  grid-row: span 2;
  background-color: lightgreen;
}

/* 3 Units High */
.tile-1-3 {
  grid-column: span 1;
  grid-row: span 3;
  background-color: lightblue;
}
.tile-2-3 {
  grid-column: span 2;
  grid-row: span 3;
  background-color: lightblue;
}
.tile-3-3 {
  grid-column: span 3;
  grid-row: span 3;
  background-color: lightblue;
}
.tile-4-3 {
  grid-column: span 4;
  grid-row: span 3;
  background-color: lightblue;
}
h1 {
  font-size: var(--font-unit);
  margin: 0;
}
h2 {
  font-size: calc(var(--font-unit) * 0.8);
  margin: 0;
}
h3 {
  font-size: calc(var(--font-unit) * 0.6);
  margin: 0;
}
h4 {
  font-size: calc(var(--font-unit) * 0.4);
  margin: 0;
}
h5 {
  font-size: calc(var(--font-unit) * 0.2);
  margin: 0;
}
h6 {
  font-size: var(--font-unit);
  margin: 0;
}
```

And we have

[[IMAGE OF LAYOUT WITH COLOR TILES AND DIFFERENT SIZES]]

## Summary of the approach

We've built a proportional layout that doesn't trigger scrollbars needlessly and behaves when a vertical scrollbar is added to the viewport.

This is accomplished with the definition of 4 CSS custom properties scoped to the `:root` element like this

``` css
:root {
  --number-of-columns: 12;
  --base-unit: calc(100% / var(--number-of-columns));
  --gap-divisor: 10;
  --gap-unit: calc(var(--base-unit) / var(--gap-divisor));
}
```

and set 5 properties on the `.layout` element like this, taking advantage of `aspect-ratio` to get what we want.

``` css
.layout {
  /* For visualization */
  background-color: grey;

  display: grid;
  grid-template-columns: repeat(12, calc(var(--base-unit) - var(--gap-unit)));
  grid-auto-rows: calc(var(--base-unit) - var(--gap-unit));
  gap: calc(var(--gap-unit) * 1.09 );
  aspect-ratio: 1 / 1;
}
```

We've done it without the use of JavaScript and without having to contend with the weirdness that is `vw`. We've leveraged `calc()` even though we can't convert between units arbitrarily.

By adding [[one]] more custom property to `:root` we're able to define a typographic system that is based on the same proportions of the the layout itself.

## What's Next?

### Breakpoints for rational tile sizing and typography

I haven't defined any breakpoints to change the width of tiles at different screenwidths.

I haven't worked out and breakpoints or a consistent typographic scheme for text elements in HTML.

### Finding some answers

There are some things I don't understand, like why the `<html>` element is drawn longer than the body or the `.layout` elements? I don't know if it matters though.

I really don't know if this breaks and what factors make it break, other than using `vw` units as values for layout measures in the `grid` definition.

### Optimizations (but not too much or many)

There might be simplifications here that make the technique more amenable to assembling a framework or design system, such as setting more variables and accomplishing more with `calc()` but I never want to be confused by what's going on.

## Join in

I started a GitHub repo[[LINK]] for this project, so feel free to use the usual tools to send in improvements or give advice.

I say this like I know what's going on, but I'm a graphic designer

[[GIF of graphic designer meme]]

not a software developer

[[GIF of IM A DOCTOR NOT A… meme]]

## Dare we dream?

A flight of fancy in totally fake CSS.

``` css
  .layout {
    display: grid;
    
    /* Set the characteristics of the grid. Is it a normal grid or proportional. If it's proportiona;l then what are we basing the proportion on? If it's width the last integer value represents columns, else it represents rows? */
    grid-basis: proportional, width, 12;
    
    /* What proportions? Use aspect-ratio */
    aspect-ratio: 1:1;
  }

  .tile-3-2 {
    /* Unitless values on width and height when assigned in the context of a proportional grid regulate column and row spans */
    width: 3;
    height: 2;
  }
```

Of course, I'm just an idea guy, somebody in some standards committee is going to have to take it and run with it.
