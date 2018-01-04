# Help

The tutorial can be done directly from the content's
[github repo UX](https://github.com/monopole/snt).

Start with the `README.md` in any directory, then
consult `README_ORDER.txt` to see the order in which to
visit remaining content.

## For a better experience

[mdrip]: https://github.com/monopole/mdrip

Download the content and the [mdrip] tool:

```
dir=$(mktemp -d)
repo=https://github.com/monopole/snt.git
git clone $repo $TST_DIR/tutorial
GOBIN=$TST_DIR go install github.com/monopole/mdrip
```

Now serve the content with `mdrip`:

```
$TST_DIR/mdrip --mode demo --port 8081 $TST_DIR/tutorial
```

Then visit http://localhost:8081 to see the material
presented with a navigation menu and one-click copying
of code blocks and check marks to show completion.

Use keys to navigate.  Hit `?` to see all key bindings.

## For an even better experience

Install and run [tmux](https://github.com/tmux/tmux/wiki),
e.g.

```
sudo apt-get install tmux
tmux
```

Then, with focus constantly in the browser, just hit
the `Enter` key to succesively execute code blocks in
tmux without using the mouse.

Use the `w` and `a` keys to skip blocks or otherwise navigate.

Hit `?` to see all bindings.
