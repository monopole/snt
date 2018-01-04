# Help

The tutorial can be done directly from the content's
[github repo UX](https://github.com/monopole/snt).

Start with the `README.md` in any directory, then
consult `README_ORDER.txt` to see the order in which to
visit remaining content.

## For a better experience

Download the content and serve it locally with
[mdrip](https://github.com/monopole/mdrip):

```
dir=$(mktemp -d)
git clone https://github.com/monopole/snt.git $dir/snt
mkdir -p $dir/bin
GOBIN=$dir/bin go install github.com/monopole/mdrip
$dir/bin/mdrip --mode demo --port 8081 $dir/snt
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

Use the `w` and `a` keys to skip blocks or otherwise navigate
(hit `?` to see all bindings).
