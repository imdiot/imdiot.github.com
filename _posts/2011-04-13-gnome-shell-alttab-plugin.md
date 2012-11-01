---
layout: post
title: "gnome-shell alt+tab 扩展"
description: ""
category: "linux"
tags: ["linux", "gnome-shell"]
---
{% include JB/setup %}

详情见《小hack一下gnome-shell的alt+tab》and《[GNOME Shell的“Alt＋Tab”革新](http://imtx.me/archives/1500.html)》

听从TualatriX建议写成了个扩展

这可是我第一个gnome-shell扩展哦

刚开始看js 也刚耍上gnome-shell

用的也是最笨的方法………………

使用方法：

在`~/.local/share/gnome-shell/extensions/`建立`alt_tab_key_press@imdiot.wordpress.com`文件夹

在`alt_tab_key_press@imdiot.wordpress.com`文件夹中建立两文件

**extension.js:**
{% highlight js linenos %}
const St = imports.gi.St;
const Mainloop = imports.mainloop;
const Shell = imports.gi.Shell;
const Clutter = imports.gi.Clutter;
const Meta = imports.gi.Meta;
const Lang = imports.lang;

const Main = imports.ui.main;
const AltTab = imports.ui.altTab;

function startAppSwitcher (shellwm, binding, window, backwards) {
    if (shellwm._workspaceSwitcherPopup != null)
        shellwm._workspaceSwitcherPopup.actor.hide();

    let tabPopup = new AltTab.AltTabPopup();
    tabPopup._keyPressEvent = function(actor, event) {
        let keysym = event.get_key_symbol();
        let event_state = Shell.get_event_state(event);
        let backwards = event_state & Clutter.ModifierType.SHIFT_MASK;
        let action = global.screen.get_display().get_keybinding_action(event.get_key_code(), event_state);

        this._disableHover();

        if (action == Meta.KeyBindingAction.SWITCH_GROUP)
            this._select(this._currentApp, backwards ? this._previousWindow() : this._nextWindow());
        else if (keysym == Clutter.Escape)
            this.destroy();
        else if (this._thumbnailsFocused) {
            if (action == Meta.KeyBindingAction.SWITCH_WINDOWS)
                if (backwards) {
                    if (this._currentWindow == 0 || this._currentWindow == -1)
                        this._select(this._previousApp());
                    else
                        this._select(this._currentApp, this._previousWindow());
                } else {
                    if (this._currentWindow == this._appIcons[this._currentApp].cachedWindows.length - 1)
                        this._select(this._nextApp());
                    else
                        this._select(this._currentApp, this._nextWindow());
                }
            else if (keysym == Clutter.Left || keysym == Clutter.a)
                this._select(this._currentApp, this._previousWindow());
            else if (keysym == Clutter.Right || keysym == Clutter.d || keysym == 96)
                this._select(this._currentApp, this._nextWindow());
            else if (keysym == Clutter.Up || keysym == Clutter.w)
                this._select(this._currentApp, null, true);
        } else {
            if (action == Meta.KeyBindingAction.SWITCH_WINDOWS)
                this._select(backwards ? this._previousApp() : this._nextApp());
            else if (keysym == Clutter.Left || keysym == Clutter.a)
                this._select(this._previousApp());
            else if (keysym == Clutter.Right || keysym == Clutter.d)
                this._select(this._nextApp());
            else if (keysym == Clutter.Down || keysym == Clutter.s || keysym == 96)
                this._select(this._currentApp, 0);
        }

        return true;
    }

    if (!tabPopup.show(backwards, binding == 'switch_group'))
        tabPopup.destroy();
}

function main() {
    Main.wm.setKeybindingHandler('switch_windows', Lang.bind(Main.wm, startAppSwitcher));
}
{% endhighlight %}

**metadata.json:**

{% highlight js linenos %}
{
    "shell-version": ["3.0.0.2"],
    "uuid": "alt_tab_key_press@imdiot.wordpress.com",
    "name": "alt_tab_key_press",
    "description": "change alt+tab key press -> begin alt+tab you can alt+w,s,a,d,` control",
    "url": "imdiot.wordpress.com"
}
{% endhighlight %}
