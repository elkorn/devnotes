[toc]
# Prerendering / prefetching
Prerendering is easy to do: just put a line of markup in __Page A__'s `
<head>
`, like so:

```
<link rel="prerender" href="/path/to/page-b/">
```

The link tells __Page A__ to load and prerender the __Page B__ while still being within __Page A__. What this means is 
that if the user navigates to __Page B__, it will display nearly instantly due to the fact that the resources have 
already been downloaded and everything was parsed in the background.

Prerendering is to be used mainly in strictly controlled flows where pages do not depend on one another. This is due 
to the fact that __Page B__ will be downloaded and prerendered with no regard to whether the user wµill navigate to it. 
Moreover, __Page B__ is downloaded while while the user is still on __Page A__- it cannot depend on any data submitted 
with __Page A__.

# CSS performance
Two important facts about CSS rule rendering costs:

**Some CSS properties are just plain more expensive to render than others.**

>This is due to the fact that they are by nature more mathematically intensive (e.g. drop-shadow)

**Combinations of CSS properties can have a greater paint time than the sum of its parts.**

> E.g. `box-shadow` alone costs around **0.22** milliseconds and `border-radius-stroke` alone about **0.27** milliseconds. But when combined, they take a whopping **1.09** milliseconds.

Another thing to note is that the `box-shadow` property itself is not the culprit. Rather, a _specific permutation of its values_ is.

>- `box-shadow: 1px 1px` takes **0.31** milliseconds
>- `box-shadow: 1px 2px 3px black` takes **0.19** milliseconds
>- `box-shadow: 1px 2px 3px 4px` takes **0.84** milliseconds

## Key points
1. Use Chrome’s _Continuous Paint_ mode in Chrome Dev Tools to get an understanding of what CSS properties are costing you.
2. Incorporate CSS reviews into your existing code review process to catch performance issues Look for places in your CSS where you are using things that are known to be more expensive, like gradients and shadows. Ask yourself, do I really need these here?
3. When in doubt, always err on the side of better performance. Your users may not remember what the padding width is on your columns, but they will remember how it feels to visit your site.
4. Avoid putting `box-shadow: 1px 2px 3px 4px` an an element that already has `border-radius:5`.

## Further reading
- [Accelerated Rendering in Chrome: The Layer Model](http://www.html5rocks.com/en/tutorials/speed/layers/)
- [GPU Accelerated Compositing in Chrome](http://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome)

# Waterfall antipatterns and techniques
[YouTube talk](http://youtu.be/O5az5D51ACQ?t=32m28s)
## Cache busting
Adding querystring parameters or a date time stamp to a resource url. Cache busting allows caching static content with a maximally long TTL.
## Tripping the lookahead tree parser

```
<meta http-equiv content-type charset>
<meta http-equiv x-ua-compatible>
<base>
```

## Stair-step pattern
If many requests are being made in a sequential order- this is due to the fact of many assets being downloaded and not enough connections available.
The resolution is to create sprite, concatenate scripts and stylesheets, shard the domains etc.