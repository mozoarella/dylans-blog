---
title: "Test page please ignore"
layout: staticpage
sitemap_exclude: true
ShowToc: true
---

# Header 1
## Header 2
### Header 3
#### Header 4
##### Header 5
###### Header 6

## Text
This is standard paragraph text, this is what it looks like **bold**, *in italics*, and with ~~strike through~~

## Callouts
{{< callout >}}
This is a default callout
{{< /callout >}}

{{< callout icon="fa-solid fa-hexagon-check fa-3x" bg_colour="#DDFFBB" fg_colour="#1572A1" ol_colour="#7882A4" >}}
Default callouts can be customized with foreground, background and outline colours. I can also set a custom icon. The outline colour defaults to the background colour if it's set, otherwise the global default is used.
{{< /callout >}}

{{< callout type="info" >}}
This is a callout of type "Info". Callouts are defined as data and also support `in line code`. Of course it also supports text that is **bold**, *in italics*, and with ~~strike through~~.
{{< /callout >}}

{{< callout type="warning" >}}
This is a callout of type "Warning". Callouts are defined as data and also support `in line code`. Of course it also supports text that is **bold**, *in italics*, and with ~~strike through~~.
{{< /callout >}}

{{< callout type="error" >}}
This is a callout of type "Error". Callouts are defined as data and also support `in line code`. Of course it also supports text that is **bold**, *in italics*, and with ~~strike through~~.
{{< /callout >}}  

{{< callout icon="fa-solid fa-heart fa-3x" bg_colour="#faf4ed" >}}
**Thanks for reading!** If you have any questions or feedback feel free to contact me through the means listed [on my main site](https://dylanmaassen.nl). Sharing my posts is also really appreciated!
{{< /callout >}}

## Lightboxes
This is a lightbox

{{< lightbox img="img/ducks.jpg" group="ducks" center="si" caption="Just a bunch of ducks" alt="Ducks" >}}

## Codeblock
Bla bla bla bla bla bla bla
```go
package main

import (
    "fmt"
)

func main() {
    fmt.Println("This is what a codeblock looks like")
}
```
Bla bla bla bla bla bla bla bla bla
