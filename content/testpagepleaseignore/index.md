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

{{< callout type="info" >}}
This is a callout of type "Info". Callouts are defined as data and also support `in line code`. Of course it also supports text that is **bold**, *in italics*, and with ~~strike through~~.
{{< /callout >}}

{{< callout type="warning" >}}
This is a callout of type "Warning". Callouts are defined as data and also support `in line code`. Of course it also supports text that is **bold**, *in italics*, and with ~~strike through~~.
{{< /callout >}}

{{< callout type="error" >}}
This is a callout of type "Error". Callouts are defined as data and also support `in line code`. Of course it also supports text that is **bold**, *in italics*, and with ~~strike through~~.
{{< /callout >}}  

## Lightbox
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
