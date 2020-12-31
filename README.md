# Image Previews and Shell Integration for **lf**

Getting-started guide to a basic `lf` setup for previews of images, documents
and video thumbnails aswell as shell integration to change the working
directory with `lf`.

![](demo.gif)

Prerequesites:
+ golang (to install from source)
+ ueberzug
+ imagemagick (for .svg previews)
+ ffmpeg (for video file thumbnails)
+ gs (for pdf previews)

## 1. Install lf

Get the source code, build and install.

```
git clone "https://github.com/gokcehan/lf" .
go install
```

Make sure `lf` can be run. Copy *$GOPATH/bin/lf* to *usr/local/bin/* if your
*$GOPATH* is not in your *$PATH*.

## 2. Install lf wrapper, preview and cleaner scripts

Install the 3 scripts *lf-run*, *lf-cleaner* and *lf-previewer* from this
repository. Make sure they're executable and in your *$PATH*.

## 3. Configure your shell

Add this function to your *.zshrc* (or *.bashrc* etc.) to set up ueberzug
previews and to give you the option of changing your working directory upon
exiting `lf`.

```
# zshrc
lf () {
	LF_TEMPDIR="$(mktemp -d -t lf-tempdir-XXXXXX)"
	LF_TEMPDIR="$LF_TEMPDIR" lf-run -last-dir-path="$LF_TEMPDIR/lastdir" "$@"
	if [ "$(cat "$LF_TEMPDIR/cdtolastdir" 2>/dev/null)" -eq 1 ]; then
		cd "$(cat "$LF_TEMPDIR/lastdir")"
	fi
	rm -r "$LF_TEMPDIR"
	unset LF_TEMPDIR
}
```

## 4. Configure lf

In your *lfrc* configure your previewer and cleaner settings and choose a
hotkey to quit `lf` and change your working directory.

```
set previewer lf-previewer
set cleaner lf-cleaner

map x quitcd
cmd quitcd ${{
	echo "1" > "$LF_TEMPDIR"/cdtolastdir
	lf -remote "send $id quit"
}}
```

Using the above, pressing *x* will change your working directory after you quit
`lf` while *q* won't (`lf`'s default behavior). If you prefer to always change
your working directory upon closing `lf` map *quitcd* to *q* instead.

Done!

---

## Further Information

Note that if you want to start `lf` from a different shell or a script you must
run `lf-run` instead of `lf`. Failing to do so will skip the proper set up of
`ueberzug` and its temporary directory and `lf`'s attempts to generate previews
will produce error messages.

The proper way to configure `lf` to work with and without the setup in *lf-run*
is to separate configurations for when `lf` was launched through `lf-run` vs.
when `lf` was launched directly. This can be done using shell blocks inside
your *lfrc*. Here's a template to get you started which will print a message
and exit immediately if `lf` is started without `lf-run`.

```
# lfrc
%{{
if [ -z "$LF_TEMPDIR" ] || [ -z "$LF_FIFO_UEBERZUG" ]; then
	# lf was launched directly without lf-run
	lf -remote "send $id \$echo 'Did you run lf instead of lf-run?'"
	lf -remote "send $id quit"
else
	# lf was launched through lf-run
fi
}}
```
