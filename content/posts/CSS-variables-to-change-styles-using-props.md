---
title: 'Use CSS Variables to style react components on demand'
date: 2024-04-14T16:40:14+10:00
draft: false
description: No css-in-js needed, no libraries, locally scoped. Perfect for creating tree-shakeable component libraries.
# used to set cover photo and open graph photo:
images: 
    - ./images/default-cover.png
slug: CSS-variables-to-change-styles-using-props
keywords:
  - css-in-js
  - css-variables
  - css
  - react
  - props  
# lastmod: 
series:
  - react
tags: 
  - javascript
  - typescript
  - react
  - css 
---



Without any adding any dependencies you can connect react props to raw css at runtime with nothing but css variables (aka "[custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)"). If you add CSS modules on top you don't have to worry about affecting the global scope so **components created in this way can be truly modular and transferrable**. I use this with [vite](https://vitejs.dev/). 

A quick rehash on how CSS variables are scoped:

```css
/* Scoped globally */
:root {
    --bright-pink: #FF3B9D;
}

/* Scoped to the container */
.containerDiv {
    --pastel-green: #A5FF8A;
}
```

----



<mark>The idea is that you lift up all the style variables to a main container class</mark> for the component. So instead of this:

```css
.someComponent {
    background-color: blue;
}
.someOtherComponent {
    background-color: red;
}
```

You do this:

```css
.containerDiv {
    --main-color: blue;
    --secondary-color: red;
}

.someComponent {
    background-color: var(--main-color);
}

.someOtherComponent {
    background-color: var(--secondary-color);
}
```



Then in your component (vite [doesn't require](https://vitejs.dev/guide/features.html#css-modules) configuring postcss to get [css modules](https://github.com/css-modules/css-modules)):

```tsx
import css from "./someCss.module.css";

interface SomePromps { 
    mainColor: string;
    secondaryColor: string;
}

export default function SomeComponent({mainColor, secondaryColor}: SomePromps) {
    
    const cssVars = {
        "--mainColor": mainColor,
        "--secondaryColor": secondaryColor,
    } as React.CSSProperties;
	
    return (
        <div className={css.containerDiv} style={cssVars}>
            // This component will automatically update anything
            // in the stylesheet that relies on the variables.
            // Props > re-render > css > display.
            <div className={css.primaryColor}></div>
            <div className={css.secondaryColor}></div>
        </div>
    )
}
```

Now changing the props will affect the css directly through the variables which will update the element's style on the page. Adding a dependency do this is unnecesary imo.

----

**Tip 1**

To reduce the number of individual color props, and to validate the arguments the end user gives, I use  the very small and 0 ext. dep. module: [colord](https://github.com/omgovich/colord) to lighten, darken, contrast, and hue rotate a couple of colors to produce more. It can even produce a pallete / collection of harmonious colors from a single argument.

**Tip 2**

See this post to create a component library using vite: https://dev.to/receter/how-to-create-a-react-component-library-using-vites-library-mode-4lma
