# About

This is a shell command tutorial written in markdown.

[mdrip]: https://github.com/monopole/mdrip
[content]: https://github.com/monopole/snt

To do the tutorial from its raw [content],
start with the `README.md` in any directory, then
consult `README_ORDER.txt` to see the order in which to
visit remaining content.

## For a better experience

Instead of working from the raw [content] at github,
serve the content locally with [mdrip]:

<!-- @serveLocally -->
```
GOBIN=$TMPDIR go install github.com/monopole/mdrip
$TMPDIR/mdrip --mode demo --port 8081 \
    https://github.com/monopole/snt
```

Visit [localhost:8081](http://localhost:8081) to see the material
presented with a navigation sidebar,
one-click copying of code blocks,
and check marks to show completion.

> _Hit `?` to see all key bindings._

## For an even better experience

On top of serving the content locally, also install and
run [tmux](https://github.com/tmux/tmux/wiki):

<!-- @installTmux -->
```
sudo apt-get install tmux
```

<!-- @runTmux -->
```
tmux
```

Then, with focus constantly in the browser, just hit
the `Enter` key to succesively execute code blocks in
`tmux` without using the mouse.

> _Hit `?` to see all key bindings._
