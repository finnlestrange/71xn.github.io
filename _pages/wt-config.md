---
layout: single
author_profile: true
permalink: /wt-config/
title: My Windows Terminal Config
---

![](/images/wt/terminal.png)

## [Oh-My-Posh Config](https://ohmyposh.dev/docs/installation/)

* Powershell profile 
{% highlight powershell linenos %}

Import-Module oh-my-posh
Set-PoshPrompt jandedobbeleer

{% endhighlight %}

## [Font](https://github.com/ryanoasis/nerd-fonts/tree/master/patched-fonts/FiraCode)

## Color Scheme

{% highlight json linenos %}

"schemes": [
        {
            "name": "Andromeda",
            "black": "#000000",
            "red": "#cd3131",
            "green": "#05bc79",
            "yellow": "#e5e512",
            "blue": "#2472c8",
            "purple": "#bc3fbc",
            "cyan": "#0fa8cd",
            "white": "#e5e5e5",
            "brightBlack": "#666666",
            "brightRed": "#cd3131",
            "brightGreen": "#05bc79",
            "brightYellow": "#e5e512",
            "brightBlue": "#2472c8",
            "brightPurple": "#bc3fbc",
            "brightCyan": "#0fa8cd",
            "brightWhite": "#e5e5e5",
            "background": "#262a33",
            "foreground": "#e5e5e5"
          }
    ],

{% endhighlight %}

## Config File

{% highlight json %}

{
    "$schema": "https://aka.ms/terminal-profiles-schema",

    "defaultProfile": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",

    "copyOnSelect": false,

    "copyFormatting": false,

    "profiles":
    {
        "defaults":
        {
            // Put settings here that you want to apply to all profiles.

            "colorScheme": "Andromeda",
            "fontFace": "FiraCode NF",
            "scrollbarState": "hidden",
            "snapOnInput": true,
            "suppressApplicationTitle": true,
            "useAcrylic": true,
            "acrylicOpacity": 0.9

        },
        "list":
        [
            {
                // Make changes here to the powershell.exe profile.
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "name": "Windows PowerShell",
                "commandline": "powershell.exe",
                "hidden": false
            },
            {
                // Make changes here to the cmd.exe profile.
                "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
                "name": "Command Prompt",
                "commandline": "cmd.exe",
                "hidden": false
            },
            {
                "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
                "hidden": true,
                "name": "Azure Cloud Shell",
                "source": "Windows.Terminal.Azure"
            }
        ]
    },

    // Add custom color schemes to this array.
    // To learn more about color schemes, visit https://aka.ms/terminal-color-schemes
    "schemes": [
        {
            "name": "Andromeda",
            "black": "#000000",
            "red": "#cd3131",
            "green": "#05bc79",
            "yellow": "#e5e512",
            "blue": "#2472c8",
            "purple": "#bc3fbc",
            "cyan": "#0fa8cd",
            "white": "#e5e5e5",
            "brightBlack": "#666666",
            "brightRed": "#cd3131",
            "brightGreen": "#05bc79",
            "brightYellow": "#e5e512",
            "brightBlue": "#2472c8",
            "brightPurple": "#bc3fbc",
            "brightCyan": "#0fa8cd",
            "brightWhite": "#e5e5e5",
            "background": "#262a33",
            "foreground": "#e5e5e5"
          }
    ],

    // Add custom actions and keybindings to this array.
    // To unbind a key combination from your defaults.json, set the command to "unbound".
    // To learn more about actions and keybindings, visit https://aka.ms/terminal-keybindings
    "actions":
    [
        // Copy and paste are bound to Ctrl+Shift+C and Ctrl+Shift+V in your defaults.json.
        // These two lines additionally bind them to Ctrl+C and Ctrl+V.
        // To learn more about selection, visit https://aka.ms/terminal-selection
        { "command": {"action": "copy", "singleLine": false }, "keys": "ctrl+c" },
        { "command": "paste", "keys": "ctrl+v" },

        // Press Ctrl+Shift+F to open the search box
        { "command": "find", "keys": "ctrl+shift+f" },

        // Press Alt+Shift+D to open a new pane.
        // - "split": "auto" makes this pane open in the direction that provides the most surface area.
        // - "splitMode": "duplicate" makes the new pane use the focused pane's profile.
        // To learn more about panes, visit https://aka.ms/terminal-panes
        { "command": { "action": "splitPane", "split": "auto", "splitMode": "duplicate" }, "keys": "alt+shift+d" }
    ]
}


{% endhighlight %}