## A specification for making configurable gtk themes

This is a way to enable gtk, cinnamon, xfwm, and GNOME Shell themes, etc. to have configurable options in a standardised way so that DEs can present the options to the user in an easy to access way.

How it works:

A theme stored in `~/.themes` can make itself configurable by including a shell script or program that will alter the theme in place according to options passed to it as arguments.

A file called `config_options.json` will be included in the theme that lists the options available.

A DE settings dialog and/or a program included in the theme directory could read the `config_options.json` file and present the available theme options to the user in a window.

When user changes options and clicks "Apply" button, the program/DE calls the configure script with the options as arguments thereby altering the theme. Program/DE then reloads the theme in the OS.

The configure script and `config_options.json` file should be in a directory called "config" in the theme directory and the script should have no external dependencies other than what would likely be found on any gtk based distro.

## Spec for the `options_config.json` file:

All fields required unless stated otherwise.
```
{
    "spec_version": <number> , Earliest version of this spec this file is compatible with. (only version=1 so far)
    "script_name": <string>, The script/program to call to apply theme changes.
    "theme_name":  <string>,
    "adwaita_link_to_gtk4": <boolean> Optional, default=false. If files (gtk.css, gtk-dark.css, assets/) in <theme>/gtk-4.0 can be linked to from ~/.config/gtk-4.0 in order to theme libadwaita apps.
    "options": [ 
            <combo_type|switch_type>, ...
        ]
}
```
combo_type. 
```
{
            "name": <string>, This is also the name of the argument passed to the script as --<name> <id_string>
            "label": <string>, 
            "type": "combo",
            "desktop": <string>, One of all|GNOME|Cinnamon|XFCE|MATE|Budgie|pop|Pantheon|Unity|<some other>. The DE in which this option should appear.
            "ids": [
                <string>, ...
            ],
            "labels": [
                <string>, ...
            ],
            "color_codes": [ Optional - use color swatches instead of labels if implemented. Color code format: "#XXXXXX"
                <string>, ...
            ],
            "value": <number> A default value with distributed theme and subsequently used to store the current user value
        }
```
switch_type.
```
{
            "name": <string>, This is also the name of the argument passed to the script when the option is selected (true) eg. --name, no argument is passed when false
            "label": <string>,
            "type": "switch",
            "desktop": <string>,
            "value": <boolean>
        }
```
TODO: Other option types could be added eg. a numerical value like Gtk.SpinButton and a color picker option.

## Behaviour of the config script:

Script should rewrite the theme in the theme directory while preserving the contained `config` directory if it exists.

Script should accept the following arguments in addition to those defined in `options_config.json`

`--name <theme name>`

The theme directory name. Create if neccessary. If not supplied, use the current parent directory of the directory containing the config script.

`--dest <path>`

Path of the folder containing the theme directory. This would normally be `~/.themes` but may also be `~/.local/share/themes` , `/usr/share/themes` or something else. Create if neccessary. If not supplied, use the current parent directory of the parent directory of the directory containing the config script.

If the theme name or path is different from its current location, the script should not copy the theme's `config` directory to the new location as this will be done by the program calling the script if wanted.

Any unrecognised arguments should be ignored.

Any missing config arguments listed in `options_config.json` should not prevent the script from completing and a default value used instead.
