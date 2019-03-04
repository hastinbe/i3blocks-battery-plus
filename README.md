battery-plus
=========
[![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)][license]

An enhanced battery status indicator for [i3blocks].

![](example.png)

## Installation

#### Requirements
* awk (POSIX compatible)
* bc
* [i3blocks]
* [upower]

#### Optional
* fonts-font-awesome
* [notify-osd], [dunst], or any [libnotify] compatible notification daemon
* [zenity]

### Guide
Copy the blocklet configuration in the given `i3blocks.conf` into your [i3blocks] configuration file, the recommended config is:

```INI
[battery-plus]
command=$SCRIPT_DIR/battery-plus
markup=pango
instance=BAT0
interval=30
#_FONT=FontAwesome
#_PERCENT=&#x25;
#_HIDE_IF_CHARGED=false
#_USE_BATT_GRADIENT=false
#_SYMBOLS_ONLY=false
#_HIDE_PERCENTAGE=false
#_HIDE_TIME_REMAINING=true
#_HIDE_TIME_TO_FULL=true
#_SHOW_CHARGE_DIRECTION=true
#_CAPACITY_ALERT=75
#_CRITICAL_ACTION=notify
#_LOW_ACTION=notify
#_NOTIFY_PROGRAM=notify-send
#_NOTIFY_EXPIRES=1500
#_NOTIFY_THROTTLE=120
#_COLOR_FULLY_CHARGED=
#_COLOR_CHARGING=yellow
#_COLOR_DISCHARGING=yellow
#_COLOR_PENDING=blue
#_COLOR_ERROR=red
#_COLOR_BATTERY=white
#_COLOR_ALERT=orange
#_COLOR_DIRECTIONAL_UP=green
#_COLOR_DIRECTIONAL_DOWN=red
#_COLOR_GRADIENT_START=#0000FF
#_COLOR_GRADIENT_END=#FF0000
#_SYMBOL_FULLY_CHARGED=
#_SYMBOL_CHARGING=&#xf0e7;
#_SYMBOL_DISCHARGING=
#_SYMBOL_UNKNOWN=&#xf128;
#_SYMBOL_PENDING=&#xf254;
#_SYMBOL_ERROR=&#xf00d;
#_SYMBOL_ALERT=&#xf071;
#_SYMBOL_BATT_100=&#xf240;
#_SYMBOL_BATT_75=&#xf241;
#_SYMBOL_BATT_50=&#xf242;
#_SYMBOL_BATT_25=&#xf243;
#_SYMBOL_BATT_0=&#xf244;
#_SYMBOL_DIRECTION_UP=&#8593;
#_SYMBOL_DIRECTION_DOWN=&#8595;
```

Set the `instance` variable to your battery to monitor. If you want to monitor multiple batteries you will need to add this blocklet twice in your i3blocks.conf with different `instance` settings.

#### Notifications
Notifications are provided by [libnotify]. Any [libnotify] compatible notification daemon can be used for notifications by specifying it with the `-n` option or the `NOTIFY_PROGRAM` variable. The most common are provided by [notify-osd] and [dunst].

Expiration time of notifications can be changed using the `-e <milliseconds>` option. Default is 1500 ms. (Ubuntu's Notify OSD and GNOME Shell both ignore the expiration parameter.)

## Customization

Designed to be fully customizable. The following can be set in your i3blocks.conf in newer versions of [i3blocks], in older versions (<= 1.4-4) you can customize using the program options. The following are configurable:

Variable | Type | Description
------------ | ------------- | -------------
_FONT | string | Font for text and symbols
_PERCENT | string | Symbol to use for the percent sign
_COLOR_FULLY_CHARGED | string | Color of the fully-charged symbol
_COLOR_CHARGING | string | Color of the charging symbol
_COLOR_DISCHARGING | string | Color of the discharging symbol
_COLOR_PENDING | string | Color of the pending charge symbol
_COLOR_ERROR | string | Color of the battery error symbol
_COLOR_BATTERY | string | Color of the battery symbol
_COLOR_ALERT | string | Color of the alert symbol
_COLOR_DIRECTIONAL_UP | string | Color of the battery charge increasing indicator
_COLOR_DIRECTIONAL_DOWN | string | Color of the battery charge decreasing indicator
_SYMBOL_FULLY_CHARGED | string | Symbol to indicate battery is fully-charged
_SYMBOL_CHARGING | string | Symbol to indicate the battery is charging
_SYMBOL_DISCHARGING | string | Symbol to indicate the battery is discharging
_SYMBOL_UNKNOWN | string | Symbol to indicate the battery state is unknown
_SYMBOL_PENDING | string | Symbol to indicate the battery state is pending
_SYMBOL_ERROR | string | Symbol to indicate the battery state is undefined
_SYMBOL_ALERT | string | Symbol to indicate there is an alert
_SYMBOL_BATT_100 | string | Symbol for a battery 100% charged
_SYMBOL_BATT_75| string | Symbol for a battery 75% charged
_SYMBOL_BATT_50 | string | Symbol for a battery 50% charged
_SYMBOL_BATT_25 | string | Symbol for a battery 25% charged
_SYMBOL_BATT_0 | string | Symbol for a battery with no/low charge
_SYMBOL_DIRECTION_UP | string | Symbol to indicate battery charge is increasing
_SYMBOL_DIRECTION_DOWN | string | Symbol to indicate battery charge is decreasing
_USE_BATT_GRADIENT | boolean | Use a color gradient for the battery symbol based on the % of charge
_SYMBOLS_ONLY | boolean | Display only symbols, no text
_HIDE_IF_CHARGED | boolean | Hide the indicator if the battery is fully-charged
_HIDE_PERCENTAGE | boolean | Hide the battery percentage
_HIDE_TIME_REMAINING | boolean | Hide the battery time remaining
_HIDE_TIME_TO_FULL | boolean | Hide the time to charge the battery to full
_SHOW_CHARGE_DIRECTION | boolean | Show the direction of change for the battery's percentage
_CAPACITY_ALERT | integer | Percentage of battery capacity to indicate an alert
_CRITICAL_ACTION | string | Action to take when the battery level reaches critical (`none`, `notify`)
_LOW_ACTION | string | Action to take when the battery level reaches low (`none`, `notify`)
_NOTIFY_PROGRAM | string | A libnotify compatible program to send notifications (`notify-send`, `dunstify`)
_NOTIFY_EXPIRES | integer | The duration, in milliseconds, for the notification to appear on screen
_NOTIFY_THROTTLE | integer | Minimum time in seconds between notifications to prevent spam

## License

`battery-plus` is released under [GNU General Public License v2][license]

Copyright (C) 1989, 1991 Free Software Foundation, Inc.

[dunst]: https://dunst-project.org
[i3blocks]: https://vivien.github.io/i3blocks/
[i3wm]: https://i3wm.org
[libnotify]: https://developer.gnome.org/libnotify
[license]: https://www.gnu.org/licenses/gpl-2.0.en.html
[notify-osd]: https://launchpad.net/notify-osd
[upower]: https://upower.freedesktop.org
[zenity]: https://wiki.gnome.org/Projects/Zenity
