## Dylans.blog

This repository contains the full source code and content for dylans.blog.

The website can be generated with the static site generator Hugo. You should use the extended version later than v0.83.0 as I make use of both SASS and WebP processing.

Now deployed through GitHub actions.

Command for local development here for my own reference `hugo server --bind 0.0.0.0 --baseURL https://blog.homedev.dy.lan --tlsAuto`

### Image generation
`./blogimagegen --title "As I was saying" --font fonts/OpenSans_SemiCondensed-Bold.ttf --output ../content/posts/as_i_was_saying/img/header.jpg --fontsize 40 --format jpeg --quality 80 --bgimg bg/sonsbeek.jpg --overlay "0,0,0,10" --frame "0,0,0,100"`

## Licensing
All of the 'source' bits such as templates, styles, scripts, configuration etc. are licensed under the MIT license.
Content, unless otherwise specified, is licensed under the Creative Commons Attribution 4.0 International License.
