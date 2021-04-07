---
layout: post
title: "Customising your keymap in X11"
date: 2021-03-28 17:00:00 +0100
categories: [misc, linux]
---

Once you start changing your keyboard layout, you start realising there's all sorts of nice improvements you can make. In X11, this used to be possible with `xmodmap`, but that's not the recommended method anymore. Here's a few quick notes on how to use the current method, `setxkbmap`, and where to find further resources.

Usually I'm happily clacking away on a 60% mechanical keyboard, but when my partner is on work calls I switch to a quieter Dell. Unlike the mech, it doesn't allow for keymaps to be set at the firmware level, so I use XKB to make modifications. It has a UK ISO layout so I use the following command:

~~~ bash
setxkbmap -model pc102 -layout gb,jp -option grp:rshift_toggle,ctrl:nocaps,altwin:swap_lalt_lwin,lv3:ralt_switch
~~~

Let's break this down:

- `setxkbmap` allows runtime configuration of XKB.
- `-model` sets the physical layout.
- `-layout` sets the keymap (to a specific country/language)
- `-option` allows for additional customizations.
- multiple arguments to the same parameter are delimited by a comma.

Some legend posted [a full list of all `setxkbmap` configurations](https://gist.github.com/jatcwang/ae3b7019f219b8cdc6798329108c9aee), which lists out all the possible options to each of these arguments.

#### Model

For model, chances are that you're rocking a generic physical layout like one of the following:

![Layouts](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b2/Physical_keyboard_layouts_comparison_ANSI_ISO_KS_ABNT_JIS.png/1920px-Physical_keyboard_layouts_comparison_ANSI_ISO_KS_ABNT_JIS.png)

Also:

- If you bought your device in Europe, chances are that yours is the 102/105 ISO layout.
- If you bought your device in the US/Canada, chances are that you have 101/104 ANSI.
- If by some chance you have a Japanese computer, you probably have the JIS layout. Lots of extra keys!

#### Layout

This will likely correspond to the language of the country where you bought your device. Note that the UK layout is under `gb`, and differs from the `us` layout in the placement of a few punctuation symbols such as `@`, `"`, `#`, `~` and `|`.

Also note that if you prefer a specific variant of a layout (e.g. Mac, Dvorak, etc.), then you can set this using the `-variant` parameter.

#### Option(s)

I love little tweaks like these. Here's the ones I have:

- `grp:rshift_toggle`: I use the right shift key to switch between UK and JP layouts.
- `ctrl:nocaps`: I don't use the caps lock, so I map the left control key to it instead. Trust me, once you make this change you won't go back.
- `altwin:swap_lalt_lwin`: Because I used i3, I use the super key (win) a lot, so it made sense to put it adjacent to the spacebar.
- `lv3:ralt_switch`: I think it makes sense to use the right alt key to access special characters such as € (although in VIM I usually stick to digraphs).


Anyways, that's all for now!

