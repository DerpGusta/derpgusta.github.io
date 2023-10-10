---
title: "Interception tools"
date: 2023-10-10
tags:
  - linux
  - keyboard
  - shortcuts
draft: false
---
I was looking for a solution to mapping the `CapsLock` key on my laptop into `Ctrl` and `Esc`. I was happy with using the setxkbmap and xcape solution for a while now, but I also had a problem with it: whenever I did a system update in Arch and the initramfs got regenerated, I would suddenly lose my bindings.

As I searched for an alternative solution, I came across [Interception Tools](https://gitlab.com/interception/linux/tools). This solution also has an additional benefit. I can use the same configuration in X, Wayland and TTY as well!

Interception tools is like middleware which helps in changing the behaviour of key events.

it contains four main CLI commands:
* `intercept` - captures the device events
* `mux` - combines multiple input event streams as one
* `udevmon` - the monitor for events
* `uinput` - redirect input events to the virtual device 

In short, we use the above tools to capture input from a device, modify it somehow (using plugins or our own executables which can handle input events) and finally sending the input to the same device again. All in the matter of a few microseconds 😄

Plugins are also available for easy configuration and remapping for common scenarios.

Here's how I did it:

0. Remove any previous configurations that modified key events I had to remove my previous settings from `.xinitrc`
```shell
# setxkbmap -option 'ctrl:nocaps'
# xcape -e 'Control_L=Escape'
```

1.  Kill the xcape process and reset the setxkbmap options
```shell
pkill xcape
setxkbmap -option
```
2. Install the tools first! I use Arch so yay does it seamlessly for me.
```shell
yay -S interception-caps2esc #installs the interception-tools (the main package) as well
```

3. Configure a job for udevmon to listen in `/etc/interception/udevmon.yaml`
```yaml
- JOB: "intercept -g $DEVNODE | caps2esc | uinput -d $DEVNODE"
  DEVICE:
    EVENTS:
      EV_KEY: [KEY_CAPSLOCK, KEY_ESC]
```

4. Enable the provided systemd service `udevmon.service`
```shell
sudo systemctl enable --now udevmon.service
```

5. Enjoy your RSI free experience!
## 🔗 Links
- [interception-tools - ArchWiki](https://wiki.archlinux.org/title/Interception-tools)
- [interception / linux / Interception Tools · GitLab](https://gitlab.com/interception/linux/tools)
- [interception / linux / plugins / caps2esc · GitLab](https://gitlab.com/interception/linux/plugins/caps2esc)
- [Virtual device - Wikipedia](https://en.wikipedia.org/wiki/Virtual_device)
- [TUTORIAL: Making Caps do both Control and Esc like Caps2Esc, but with only hyprland+ydotool : hyprland](https://old.reddit.com/r/hyprland/comments/11v6iif/tutorial_making_caps_do_both_control_and_esc_like/)
