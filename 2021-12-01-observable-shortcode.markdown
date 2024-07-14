---
date: 2021-12-01T12:00:00Z
title: Observable Hugo Shortcode
url: /observable-hugo-shortcode
---

Here's a shortcode for embedding Observable notebooks in Hugo.

```
{{</* obs
    specifier="@ballingt/embedding-example"
    cells="vegaPetalsWidget,viewof minSepalLength,viewof minSepalWidth"
*/>}}
```
When the shortcode is installed in layouts/shortcodes/obs.html in your Hugo site, the input above produces the output below.

{{< obs
    specifier="@ballingt/embedding-example"
    cells="vegaPetalsWidget,viewof minSepalLength,viewof minSepalWidth"
>}}

Here's the code to stick in the file layouts/shortcodes/obs.html.

```html
<iframe
  width="100%"
  frameborder="0"
  src="https://observablehq.com/embed/{{
    .Get "specifier"
  }}{{ with .Get "cells"
    }}?cells={{ . }}{{
  end }}"></iframe>
<script>
(function() {
 let s = document.querySelectorAll('script');
 let script = s[s.length - 1];
 let iframe = script.previousElementSibling;
 window.addEventListener("message", (e) => {
  if (e.origin === 'https://observablehq.com'
      && e.source === iframe.contentWindow) {
   const msg = JSON.parse(e.data)
   if (msg.context === 'iframe.resize') {
    iframe.height = msg.height;
   }
  }
 });
})();
</script>
```
