#!/usr/bin/env bash
#
#  battery-plus
#
#  An enhanced battery status indicator for i3blocks.
#
#  Requires:
#    awk (POSIX compatible)
#    bc
#    upower
#
#  Recommended:
#    fonts-font-awesome or fonts-hack
#
#  Optional:
#    notify-send or dunstify -- for notifications
#    zenity -- for dialogs
#
#  Copyright (c) 2018 Beau Hastings. All rights reserved.
#  License: GNU General Public License v2
#
#  Author: Beau Hastings <beau@saweet.net>
#  URL: https://github.com/hastinbe/i3blocks-battery-plus

_PERCENT="&#x25;"

# Hide the battery status if fully-charged,
_HIDE_IF_CHARGED=${_HIDE_IF_CHARGED:-false}

# Hide if battery is about to be charged and over than this value
_HIDE_OVER_PERCENTAGE=${_HIDE_OVER_PERCENTAGE:-100}

# Color the battery symbol using gradient.
_USE_BATT_GRADIENT=${_USE_BATT_GRADIENT:-false}

# Only show symbols.
_SYMBOLS_ONLY=${_SYMBOLS_ONLY:-false}

# Don't use any symbols.
_SUPPRESS_SYMBOLS=${_SUPPRESS_SYMBOLS:-false}

# Hide the battery percentage
_HIDE_PERCENTAGE=${_HIDE_PERCENTAGE:-false}

# Hide the battery time remaining
_HIDE_TIME_REMAINING=${_HIDE_TIME_REMAINING:-true}

# Hide the time to charge the battery to full
_HIDE_TIME_TO_FULL=${_HIDE_TIME_TO_FULL:-true}

# Show the direction of change for the battery's percentage
_SHOW_CHARGE_DIRECTION=${_SHOW_CHARGE_DIRECTION:-true}

# Show an alert symbol when the battery capacity drops to the given percent (0=disable).
_CAPACITY_ALERT=${_CAPACITY_ALERT:-75}

# Action to take when the battery level reaches critical.
_CRITICAL_ACTION=${_CRITICAL_ACTION:-"notify"}

# Action to take when the battery level reaches low.
_LOW_ACTION=${_LOW_ACTION:-"notify"}

# Method to use for notifications
_NOTIFY_PROGRAM=${_NOTIFY_PROGRAM:-"notify-send"}

# The duration, in milliseconds, for the notification to appear on screen.
_NOTIFY_EXPIRES="1500"

# Minimum time in seconds between notifications to prevent spam.
_NOTIFY_THROTTLE=${_NOTIFY_THROTTLE:-120}

# Colors
_COLOR_FULLY_CHARGED=${_COLOR_FULLY_CHARGED:-""}
_COLOR_CHARGING=${_COLOR_CHARGING:-"yellow"}
_COLOR_DISCHARGING=${_COLOR_CHARGING:-"yellow"}
_COLOR_PENDING=${_COLOR_PENDING:-"blue"}
_COLOR_ERROR=${_COLOR_ERROR:-"red"}
_COLOR_BATTERY=${_COLOR_BATTERY:-"white"}
_COLOR_ALERT=${_COLOR_ALERT:-"orange"}
_COLOR_DIRECTIONAL_UP=${_COLOR_DIRECTIONAL_UP:-"green"}
_COLOR_DIRECTIONAL_DOWN=${_COLOR_DIRECTIONAL_DOWN:-"red"}
_COLOR_GRADIENT_START=${_COLOR_GRADIENT_START:-"#FF0000"}
_COLOR_GRADIENT_END=${_COLOR_GRADIENT_END:-"#00FF00"}

# Symbols
if ! $_SUPPRESS_SYMBOLS; then
    _SYMBOL_FULLY_CHARGED=${_SYMBOL_FULLY_CHARGED:-""}
    _SYMBOL_CHARGING=${_SYMBOL_CHARGING:-"&#xf0e7;"}
    _SYMBOL_DISCHARGING=${_SYMBOL_DISCHARGING:-""}
    _SYMBOL_UNKNOWN=${_SYMBOL_UNKNOWN:-"&#xf128;"}
    _SYMBOL_PENDING=${_SYMBOL_PENDING:-"&#xf254;"}
    _SYMBOL_ERROR=${_SYMBOL_ERROR:-"&#xf00d;"}
    _SYMBOL_ALERT=${_SYMBOL_ALERT:-"&#xf071;"}
    _SYMBOL_BATT_100=${_SYMBOL_BATT_100:-"&#xf240;"}
    _SYMBOL_BATT_75=${_SYMBOL_BATT_75:-"&#xf241;"}
    _SYMBOL_BATT_50=${_SYMBOL_BATT_50:-"&#xf242;"}
    _SYMBOL_BATT_25=${_SYMBOL_BATT_25:-"&#xf243;"}
    _SYMBOL_BATT_0=${_SYMBOL_BATT_0:-"&#xf244;"}
    _SYMBOL_DIRECTION_UP=${_SYMBOL_DIRECTION_UP:-"&#8593;"}
    _SYMBOL_DIRECTION_DOWN=${_SYMBOL_DIRECTION_DOWN:-"&#8595;"}
fi

empty() {
    [[ -z $1 ]]
}

not_empty() {
    [[ -n $1 ]]
}

# Get a the system's temporary files directory.
#
# Notes:
#   Executes a dry-run so we don't create a file.
#   Linux doesn't require the template option, but MacOSX does.
#
# Returns:
#   The path to the temporary directory.
get_temp_dir() {
    dirname "$(mktemp -ut "battery-plus.XXX")"
}

# Output text wrapped in a span element
#
# Options:
#   -c <color>  To specify a text color
#
# Arguments:
#   $1 or $3 - String to encapsulate within a span
span() {
    local -A attribs
    local text="$*"

    if not_empty "$FONT"; then attribs[font]="$FONT"; fi

    if [ "$1" = "-c" ]; then
        if not_empty "$2"; then
            # shellcheck disable=SC2034
            attribs[color]="$2"
        fi
        text="${*:3}"
    fi

    echo "<span$(build_attribs attribs)>$text</span>"
}

# Builds html element attributes
#
# Arguments:
#   $1 - An associative array of attribute-value pairs
build_attribs() {
    local -n array=$1
    for k in "${!array[@]}"; do
        echo -n " $k='${array[$k]}'"
    done
    echo
}

# Interpolate between 2 RGB colors
#
# Arguments:
#   $1 - Color to interpolate from, as a hex triplet prefixed with #
#   $2 - Color to interpolate to, as a hex triplet prefixed with #
#   $3 - Amount of steps needed to get from start to end color
interpolate_rgb() {
    local -i steps=$3
    local -a colors
    local color1=$1
    local color2=$2
    local reversed=false

    if (( 0x${color1:1:5} > 0x${color2:1:5} )); then
        color1=$2
        color2=$1
        reversed=true
    fi

    printf -v startR "%d" "0x${color1:1:2}"
    printf -v startG "%d" "0x${color1:3:2}"
    printf -v startB "%d" "0x${color1:5:2}"
    printf -v endR "%d" "0x${color2:1:2}"
    printf -v endG "%d" "0x${color2:3:2}"
    printf -v endB "%d" "0x${color2:5:2}"

    stepR=$(( (endR - startR) / steps ))
    stepG=$(( (endG - startG) / steps ))
    stepB=$(( (endB - startB) / steps ))

for i in $(seq 0 "$steps"); do
        local -i R=$(( startR + (stepR * i) ))
        local -i G=$(( startG + (stepG * i) ))
        local -i B=$(( startB + (stepB * i) ))

        colors+=( "$(printf "#%02x%02x%02x\n" $R $G $B)" )
    done
    colors+=( "$(printf "#%02x%02x%02x\n" "$endR" "$endG" "$endB")" )

    if $reversed; then
        reverse colors colors_reversed
        # shellcheck disable=SC2154
        echo "${colors_reversed[@]}"
    else
        echo "${colors[@]}"
    fi
}

# Reverse an array
#
# Arguments:
#   $1 - Array to reverse
#   $2 - Destination array
reverse() {
    local -n arr="$1" rev="$2"
    for i in "${arr[@]}"
    do
        rev=("$i" "${rev[@]}")
    done
}

min() {
    echo "if ($1 < $2) $1 else $2" | bc
}

max() {
    echo "if ($1 > $2) $1 else $2" | bc
}

clamp() {
    min "$(max "$1" "$2")" "$3"
}

# Retrieve an item from the provided lookup table based on the percentage of a number
#
# Arguments:
#   $1 - A number
#   $2 - Number to get a percentage of
#   $3 - An array of potential values
plookup() {
    if empty "$2" || empty "$3"; then
        return
    fi

    local table
    IFS=' ' read -ra table <<< "${@:3}"

    local number1
    local number2=${2%.*}

    number1=$(min "${1%.*}" "$number2")

    local percent_max=$(( number1 * number2 / 100 ))
    local index=$(( percent_max * ${#table[@]} / 100 ))

    index=$(clamp $index 0 $(( ${#table[@]} - 1 )) )

    echo "${table[$index]}"
}

# Get battery status.
#
# Returns:
#   The battery's status or state.
get_battery_status() {
    echo "$__UPOWER_INFO" | awk -W posix '$1 == "state:" {print $2}'
}

# Get battery warning level.
#
# Returns:
#   The battery's warning level.
get_battery_warning_level() {
    echo "$__UPOWER_INFO" | awk -W posix '$1 == "warning-level:" {print $2}'
}

# Get battery percentage.
#
# Returns:
#   The battery's percentage, without the %.
get_battery_percent() {
    echo "$__UPOWER_INFO" | awk -W posix '$1 == "percentage:" { gsub("%","",$2); print $2}'
}

# Get battery capacity.
#
# Returns:
#   The battery's capcity, without the %.
get_battery_capacity() {
    echo "$__UPOWER_INFO" | awk -W posix '$1 == "capacity:" { gsub("%","",$2); print $2}'
}

# Get battery time left.
#
# Returns:
#   The battery's time left.
get_time_to_empty() {
    echo "$__UPOWER_INFO" | awk -W posix -F':' '/time to empty:/{print $2}' | xargs
}

# Get the time remaining until the battery is fully charged.
#
# Returns:
#   The time remaining until battery is fully charged.
get_time_to_full() {
    echo "$__UPOWER_INFO" | awk -W posix -F':' '/time to full:/{print $2}' | xargs
}

# Get symbol for the given battery percentage.
#
# Arguments:
#   $percent  (int)  Battery percentage.
#
# Returns:
#  The battery symbol.
get_battery_charge_symbol() {
    local percent="$1"
    local symbol

    if (( percent >= 90 )); then symbol="$_SYMBOL_BATT_100"
    elif (( percent >= 70 )); then symbol="$_SYMBOL_BATT_75"
    elif (( percent >= 40 )); then symbol="$_SYMBOL_BATT_50"
    elif (( percent >= 20 )); then symbol="$_SYMBOL_BATT_25"
    else symbol="$_SYMBOL_BATT_0"
    fi

    echo "$symbol"
}

# Get battery status symbol.
#
# Returns:
#   An symbol name, following the Symbol Naming Specification
get_battery_status_symbol() {
    echo "$__UPOWER_INFO" | awk -W posix '$1 == "symbol-name:" {print $2}'
}

# Get color for the given battery percentage.
#
# Arguments:
#   $percent  (int)  Battery percentage.
#
# Returns:
#   The color to use for the given battery percentage.
get_percentage_color() {
    local colors
    read -ra colors <<< "$(interpolate_rgb "$_COLOR_GRADIENT_START" "$_COLOR_GRADIENT_END" 8)"
    plookup "$1" 100 "${colors[@]}"
}

# Determines whether or not we can send a notification.
#
# Returns:
#   0 on false, 1 on true.
can_notify() {
    local -i now
    local -i last
    local -i diff

    now=$(date +"%s")
    last=$(get_last_notify_time)

    if empty "$last"; then return 0; fi

    diff=$(( now - last ))

    (( diff > _NOTIFY_THROTTLE ))
}

# Get the last time we sent a notification.
#
# Returns:
#   Time in seconds since the Epoch.
get_last_notify_time() {
    if [ -f "$TMP_NOTIFY" ]; then
        stat -c '%Y' "$TMP_NOTIFY"
    fi
}

# Save the last time we sent a notification.
save_last_notify_time() {
    touch -m "$TMP_NOTIFY"
}

# Display the charging indicator.
#
# Returns:
#   The charging indicator.
display_batt_charging() {
    if not_empty "$_SYMBOL_CHARGING"; then
        echo -n "$(span -c "${_COLOR_CHARGING}" "$_SYMBOL_CHARGING") "
    fi

    echo "$colored_battery_symbol"
}

# Display the discharging indicator.
#
# Returns:
#   The discharging indicator.
display_batt_discharging() {
    if not_empty "$_SYMBOL_DISCHARGING"; then
        echo -n "$(span -c "${_COLOR_DISCHARGING}" "$_SYMBOL_DISCHARGING") "
    fi

    echo "$battery_charge_symbol"
}

# Display the pending charge indicator.
#
# Returns:
#   The pending charge indicator.
display_batt_pending_charge() {
    if not_empty "$_SYMBOL_PENDING"; then
       echo -n "$(span -c "${_COLOR_PENDING}" "$_SYMBOL_PENDING") "
    fi

    if not_empty "$_SYMBOL_CHARGING"; then
        echo -n "$(span -c "${_COLOR_CHARGING}" "$_SYMBOL_CHARGING") "
    fi

    echo "${battery_charge_symbol}"
}

# Display the pending discharge indicator.
#
# Returns:
#   The pending discharge indicator.
display_batt_pending_discharge() {
    if not_empty "$_SYMBOL_PENDING"; then
        echo -n "$(span -c "${_COLOR_PENDING}" "$_SYMBOL_PENDING") "
    fi

    if not_empty "$_SYMBOL_DISCHARGING"; then
        echo -n "$(span -c "${_COLOR_DISCHARGING}" "$_SYMBOL_DISCHARGING") "
    fi

    echo "$colored_battery_symbol"
}

# Display the empty battery indicator.
#
# Returns:
#   The empty battery indicator.
display_batt_empty() {
    echo "$battery_charge_symbol"
}

# Display the fully charged indicator.
#
# Returns:
#   The fully-charged indicator.
display_batt_fully_charged() {
   if not_empty "$_SYMBOL_FULLY_CHARGED"; then
        echo -n "$(span -c "${_COLOR_FULLY_CHARGED}" "$_SYMBOL_FULLY_CHARGED") "
    fi

    echo "$battery_charge_symbol"
}

# Display the unknown battery indicator.
#
# Returns:
#   The unknonw battery indicator.
display_batt_unknown() {
    if not_empty "$_SYMBOL_UNKNOWN"; then
        echo -n "$(span "$_SYMBOL_UNKNOWN") "
    fi

    get_battery_charge_symbol 0
}

# Display the battery error indicator.
#
# Returns:
#   The battery error indicator.
display_batt_error() {
    span -c "${_COLOR_ERROR}" "$_SYMBOL_ERROR" "$_SYMBOL_BATT_0"
}

# Display the battery percentage.
#
# Returns:
#   The battery percentage, or nothing if disabled.
display_percentage() {
    if [[ "$_SYMBOLS_ONLY" = false && "$_HIDE_PERCENTAGE" = false ]]; then
        echo -n " $(span -c "${percentage_color}" "${battery_percentage:---}${_PERCENT}")"

        if [ "$status" = "charging" ] && $_SHOW_CHARGE_DIRECTION; then
            echo -n "$(span -c "${_COLOR_DIRECTIONAL_UP}" "${_SYMBOL_DIRECTION_UP}")"
        elif [ "$status" = "discharging" ] && $_SHOW_CHARGE_DIRECTION; then
            echo -n "$(span -c "${_COLOR_DIRECTIONAL_DOWN}" "${_SYMBOL_DIRECTION_DOWN}")"
        fi

        echo
    fi
}

# Display the battery time remaining.
#
# Arguments:
#   $force  (bool) Force display.
#
# Returns:
#   The time remaining, or nothing if disabled.
display_time_remaining() {
    local force=${1:-false}

    if $force || ! [[ "$_SYMBOLS_ONLY" = true || "$_HIDE_TIME_REMAINING" = true ]]; then
        time_to_empty=$(get_time_to_empty)
        if not_empty "$time_to_empty"; then
            echo "$time_to_empty"
        fi
    fi
}

# Display the battery time to fully charge.
#
# Arguments:
#   $force  (bool) Force display.
#
# Returns:
#   The time to fully charge, or nothing if disabled.
display_time_to_full() {
    local force=${1:-false}

    if $force || ! [[ "$_SYMBOLS_ONLY" = true || "$_HIDE_TIME_TO_FULL" = true ]]; then
        time_to_full=$(get_time_to_full)
        if not_empty "$time_to_full"; then
            echo "$time_to_full"
        fi
    fi
}

# Displays an array of segments.
#
# Arguments:
#   $segments  (array)  The an array of segments.
#
# Returns:
#   Each segment after another, followed by a newline.
display_segments() {
    local -a segments=("$@")

    for segment in "${segments[@]}"; do
        if not_empty "$segment"; then
            echo -n "$segment"
        fi
    done

    echo
}

# Colors text using either the first or second color
#
# Arguments:
#   $1 - Text
#   $2 - Color to use if toggle is false
#   $3 - Color to use if toggle is true
#   $4 - A boolean used to toggle between colors
#
# Returns:
#   The colored text
multicolor() {
    local color="$2"
    local toggle=$4

    if $toggle && not_empty "$3"; then
        color="$3"
    fi

    if not_empty "$color"; then
        span -c "$color" "$1"
    else
        span "$1"
    fi
}


# Display a notification.
#
# Arguments:
#   $text  (string)  The notification text.
#   $symbol  (string)  Name of an symbol.
#   $type  (string)  The type of notification.
#   $force (string)  Force the notification, ignoring any throttle.
#   $rest  (string)  Any additional options to pass to the command.
notify() {
    local text="$1"
    local symbol="$2"
    local type="$3"
    local force="$4"
    local rest="${*:5}"
    local command

    if empty "$text" || ! $force && ! can_notify; then return; fi

    if [ "$_NOTIFY_PROGRAM" = "dunstify" ]; then
        command="dunstify -r 1001"
    elif [ "$_NOTIFY_PROGRAM" = "notify-send" ]; then
        command="notify-send"
    fi

    if not_empty "$_NOTIFY_EXPIRES"; then command="$command -t $_NOTIFY_EXPIRES"; fi
    if not_empty "$symbol"; then command="$command -i $symbol"; fi
    if not_empty "$type"; then command="$command -u $type"; fi

    command="$command $rest \"$text\""
    if eval "$command"; then
        save_last_notify_time
    fi
}

# Display a dialog.
#
# Arguments:
#   $text  (string)  The dialog text.
#   $symbol  (string)  Name of an symbol.
#   $type  (string)  The type of dialog.
#   $rest  (string)  Any additional options to pass to the command.
dialog() {
    local text="$1"
    local symbol="$2"
    local type="$3"
    local rest="${*:4}"

    if empty "$text"; then
        return
    fi

    command="zenity"

    if not_empty "$symbol"; then command="$command --icon-name=\"$symbol\""; fi
    if not_empty "$type"; then command="$command --${type}"; fi

    command="$command --text=\"$text\" $rest"
    eval "$command"
}

# Display program usage.
usage() {
    echo -e "Usage: $0 [options]
Display the battery status using upower for i3blocks.\n
Options:
  -a <none|notify>\taction to take when battery is low
  -A <color>\t\tcolor of the alert symbol
  -B <color>\t\tcolor of the battery symbol
  -c <capacity>\t\tset battery capacity alert percentage (0=disable)
  -C <color>\t\tcolor of the charging symbol
  -d\t\t\tshow the direction of change for the battery's percentage
  -D <color>\t\tcolor of the discharging symbol
  -e <millseconds>\texpiration time of notifications (Ubuntu's Notify OSD and GNOME Shell both ignore this parameter.)
  -E <color>\t\tcolor of the battery error symbol
  -f <font>\t\tfont for text and symbols
  -F <color>\t\tcolor of the fully-charged symbol
  -G\t\t\tcolor the battery symbol according to battery percentage
  -h\t\t\tdisplay this help and exit
  -H\t\t\tsuppress displaying if battery is fully-charged
  -i <color>\t\tcolor gradient start color
  -I\t\t\tonly display symbols
  -j <color>\t\tcolor gradient end color
  -J <symbol>\t\tsymbol for a battery 100% charged
  -k\t\t\thide the battery percentage
  -K <symbol>\t\tsymbol for a battery 75% charged
  -l <none|notify>\taction to take when the battery level reaches critical
  -L <symbol>\t\tsymbol to indicate there is an alert
  -M <symbol>\t\tsymbol for a battery 50% charged
  -n <program>\t\ta libnotify compatible notification program
  -N <color>\t\tcolor of the battery charge decreasing indicator
  -o <percentage>\tpercentage above which a battery is considered full
  -O <symbol>\t\tsymbol for a battery with no/low charge
  -p <symbol>\t\tsymbol to use for the percent sign
  -P <color>\t\tcolor of the pending charge symbol
  -Q <symbol>\t\tsymbol for a battery 25% charged
  -r\t\t\thide the battery time remaining
  -R <symbol>\t\tsymbol to indicate the battery state is undefined
  -s\t\t\tsuppress symbols
  -S <symbol>\t\tsymbol to indicate battery is fully charged
  -t <seconds>\t\tminimum time in seconds between notifications to prevent spam
  -T <symbol>\t\tsymbol to indicate the battery is charging
  -u\t\t\thide the time to charge the battery to full
  -U <color>\t\tcolor of the battery charge increasing indicator
  -V <symbol>\t\tsymbol to indicate the battery state is unknown
  -W <symbol>\t\tsymbol to indicate battery charge is increasing
  -X <symbol>\t\tsymbol to indicate the battery state is pending
  -Y <symbol>\t\tsymbol to indicate the battery is discharging
  -Z <symbol>\t\tsymbol to indicate battery charge is decreasing
" 1>&2
    exit 1
}

###############################################################################

declare -a long_segments
declare -a short_segments

__UPOWER_INFO=$(upower --show-info "/org/freedesktop/UPower/devices/battery_${BLOCK_INSTANCE:-BAT0}")
TMP_NOTIFY="$(get_temp_dir)/battery-plus.last_notify"

while getopts "a:A:B:c:C:dD:e:E:F:GhHi:Ij:J:kK:l:L:M:n:N:o:O:p:P:Q:rR:sS:t:T:uU:V:W:X:Y:Z:" o; do
    case "$o" in
        a) _LOW_ACTION="${OPTARG}" ;;
        A) _COLOR_ALERT="${OPTARG}" ;;
        B) _COLOR_BATTERY="${OPTARG}" ;;
        c) _CAPACITY_ALERT="${OPTARG}" ;;
        C) _COLOR_CHARGING="${OPTARG}" ;;
        d) _SHOW_CHARGE_DIRECTION=true ;;
        D) _COLOR_DISCHARGING="${OPTARG}" ;;
        e) _NOTIFY_EXPIRES="${OPTARG}" ;;
        E) _COLOR_ERROR="${OPTARG}" ;;
        F) _COLOR_FULLY_CHARGED="${OPTARG}" ;;
        G) _USE_BATT_GRADIENT=true ;;
        H) _HIDE_IF_CHARGED=true ;;
        i) _COLOR_GRADIENT_START="${OPTARG}" ;;
        I) _SYMBOLS_ONLY=true ;;
        j) _COLOR_GRADIENT_END="${OPTARG}" ;;
        J) _SYMBOL_BATT_100="${OPTARG}" ;;
        k) _HIDE_PERCENTAGE=true ;;
        K) _SYMBOL_BATT_75="${OPTARG}" ;;
        l) _CRITICAL_ACTION="${OPTARG}" ;;
        L) _SYMBOL_ALERT="${OPTARG}" ;;
        M) _SYMBOL_BATT_50="${OPTARG}" ;;
        n) _NOTIFY_PROGRAM="${OPTARG}" ;;
        N) _COLOR_DIRECTIONAL_DOWN="${OPTARG}" ;;
        o) _HIDE_OVER_PERCENTAGE="${OPTARG}" ;;
        O) _SYMBOL_BATT_0="${OPTARG}" ;;
        p) _PERCENT="${OPTARG}" ;;
        P) _COLOR_PENDING="${OPTARG}" ;;
        Q) _SYMBOL_BATT_25="${OPTARG}" ;;
        r) _HIDE_TIME_REMAINING=true ;;
        R) _SYMBOL_ERROR="${OPTARG}" ;;
        s) _SUPPRESS_SYMBOLS=true ;;
        S) _SYMBOL_FULLY_CHARGED="${OPTARG}" ;;
        t) _NOTIFY_THROTTLE="${OPTARG}" ;;
        T) _SYMBOL_CHARGING="${OPTARG}" ;;
        u) _HIDE_TIME_TO_FULL=true ;;
        U) _COLOR_DIRECTIONAL_UP="${OPTARG}" ;;
        V) _SYMBOL_UNKNOWN="${OPTARG}" ;;
        W) _SYMBOL_DIRECTION_UP="${OPTARG}" ;;
        X) _SYMBOL_PENDING="${OPTARG}" ;;
        Y) _SYMBOL_DISCHARGING="${OPTARG}" ;;
        Z) _SYMBOL_DIRECTION_DOWN="${OPTARG}" ;;
        h | *) usage ;;
    esac
done
shift $((OPTIND-1)) # Shift off options and optional --

battery_percentage=$(get_battery_percent)
percentage_color=$(get_percentage_color "$battery_percentage")
capacity=$(get_battery_capacity)
battery_charge_symbol=$(get_battery_charge_symbol "$battery_percentage")
battery_status_symbol=$(get_battery_status_symbol)
colored_battery_symbol=$(multicolor "$battery_charge_symbol" "$_COLOR_BATTERY" "$percentage_color" $_USE_BATT_GRADIENT)
warning_level=$(get_battery_warning_level)

# Handle battery warning levels
case "$warning_level" in
    critical)
        case "$_CRITICAL_ACTION" in
            notify) notify "$(span "Your battery level is ${warning_level}!")" "$battery_status_symbol" "critical" false "-c device" ;;
        esac
        ;;
    low)
        case "$_LOW_ACTION" in
            notify) notify "$(span "Your battery level is ${warning_level}")" "$battery_status_symbol" "normal" false "-c device" ;;
        esac
        ;;
esac

# Displayable alerts
if (( _CAPACITY_ALERT > 0 && "${capacity%.*}" <= _CAPACITY_ALERT )); then
    CAPACITY_TRIGGERED=true
    long_segments+=( "$(span -c "$_COLOR_ALERT" "$_SYMBOL_ALERT") " )
fi

# Battery statuses
status=$(get_battery_status)
case "$status" in
    charging)           long_segments+=( "$(display_batt_charging)" ) ;;
    discharging)        long_segments+=( "$(display_batt_discharging)" ) ;;
    empty)              long_segments+=( "$(display_batt_empty)" ) ;;
    fully-charged)
        if $_HIDE_IF_CHARGED; then
            exit 0
        fi
        long_segments+=( "$(display_batt_fully_charged)" )
        ;;
    pending-charge)
        if $_HIDE_IF_CHARGED && (( battery_percentage >= _HIDE_OVER_PERCENTAGE )); then
            exit 0
        fi
        long_segments+=( "$(display_batt_pending_charge)" )
        ;;
    pending-discharge)  long_segments+=( "$(display_batt_pending_discharge)" ) ;;
    unknown)            long_segments+=( "$(display_batt_unknown)" ) ;;
    *)                  long_segments+=( "$(display_batt_error)" ) ;;
esac

# Since long_segments contains no text at this point, lets use it for the short_text
short_segments=( "${long_segments[@]}" )

# Append all text segments
long_segments+=( "$(display_percentage)" )
if not_empty "$(display_time_remaining)"; then
    long_segments+=( "($(display_time_remaining))" )
fi
if not_empty "$(display_time_to_full)"; then
    long_segments+=( "($(display_time_to_full))" )
fi

# Display the block long_text
display_segments "${long_segments[@]}"

# Display the block short_text
display_segments "${short_segments[@]}"

# Handle click events
case $BLOCK_BUTTON in
    1)
        if not_empty "$CAPACITY_TRIGGERED"; then
            dialog "$(span "Your battery capacity has reduced to ${capacity%.*}${_PERCENT}!")" "battery-caution" "warning" "--no-wrap"
        else
            if [ "$status" = "discharging" ]; then
                notify "$(span "$(display_time_remaining true) remaining")" "alarm-symbolic" "normal" true "-c device"
            elif [ "$status" = "charging" ]; then
                notify "$(span "$(display_time_to_full true) until fully charged")" "alarm-symbolic" "normal" true "-c device"
            fi
        fi
        ;;
esac

if not_empty "$battery_percentage" && (( battery_percentage <= 5 )); then
    exit 33
fi
