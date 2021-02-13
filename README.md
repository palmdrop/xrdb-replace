<!-- PROJECT LOGO -->
<br />
<p align="center">
  <h1 align="center">xrdb-replace</h1>

  <p align="center">
    A script for generating config files using templates and xrdb: perfect for applications that disallows configuration through Xresources.
    <br />
  </p>
</p>

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table of Contents</h2></summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li>
      <a href="#usage">Usage</a>
      <ul>
        <li><a href="#basic usage">Basic usage</a></li>
        <li><a href="#reload script">Reload script</a></li>
      </ul>
    </li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact and social media</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## About The Project
Being able to customize the look-and-feel of various applications in a single place is extremely convenient. For many Linux-users, that place is Xresources. However, not all application read variables using the xrdb database. Instead, these applications have to be configured purely using their individual config files, which can be a hassle if you e.g want to achieve a coherent colorscheme across your system. 

This script resolves this issue. Create a template of your config file and indicate whever you wish to read values from the xrdb database. Run the script, and a new config file will be produced with the specified values. 

Every time you edit Xresources or make changes in your template (which you should edit now, instead of your regular config file), run this script for your new changes to take effect.

Some applications, require restart for new settings to be used. The script also has support for running a reload script written by the user, which could be configured to restart the affected applications.

More under <a href="#usage">usage</a>.

<!-- GETTING STARTED -->
## Getting Started
This is just a simple bash script. Getting started shouldn't be that complicated. To use the script correctly, please read <a href="#usage">usage</a>.

### Prerequisites
I can't imagine that any of these utilities are not already part of your system, but here's a short list:

* xrdb
* grep
* sort
* sed
* cut

(Unfortunately, the script is not POSIX-compliant partly due to the use of arrays.)

### Installation

1. Clone the repo
   ```sh
   git clone https://github.com/palmdrop/xrdb-replace.git
   ```
2. Enter the cloned repository
   ```sh
   cd xrdb-replace
   ```
3. Move the `xrdb-replace` file to somewhere within your path, e.g
   ```sh
   mv xrdb-replace /usr/bin/xrdb-replace
   ```

<!-- USAGE EXAMPLES -->
## Usage

This message will be displayed when running `xrdb-replace --help`:

```sh
usage: $(basename $0) [-options ...] [-f file_path] [-d delimiter] [-r reload_script] [target_file] [prefix]
    -F, --from-default          Read from default settings file. Usually ~/.config/xrdb-replace/files
    -q, --quote                 Surround xrdb values with quotes in "target_file"
    -R, --reload-default        Run default reload script. Should be located in the same directory as "target_file" and named "reload.sh"
    -g, --glob                  Use a glob ("*") instead of "prefix" if a specific xrdb value cannot be found
    -b, --backup                Create a backup of "target_file", named "target_file.bak"
    -f, --from-file file_path   Use "file_path" as settings file.
    -d, --delimiter delimiter   Indicates that "delimiter" surrounds every xrdb variable in the template file
    -r, --reload reload_script  Specifies the name of the reload script to be used
    -h, --help                  Prints this message
    target_file                 The target file to apply the script to. Only used if "-f" or "-F" is not passed
    prefix                      The xrdb prefix used to identify xrdb variables in the template
```

### Basic usage
An example of basic usage:

```sh
xrdb-replace ~/.config/app/apprc app
```

This tells the script to look for a template in the same directory as `apprc`. The template should be called `apprc.in`. The template should be identical to your regular config file, with the exception of the values you want to read from the xrdb database. These should be prefixed with `app` and surrounded by `%`.

If this is your regular `apprc`

```sh
# ~/.config/app/apprc
set background #222222
set foreground #dddddd
set borderpx   10
```

this is a possible template:

```sh
# ~/.config/app/apprc.in
set background %app.background%
set foreground %app.foreground%
set borderpx   %app.borderpx%
```

The delimiter `%` prevents the script from matching unwanted values. For some config files, this might not be optimal. The `-d` flag can be used to change the delimiter. 

After trunning the script, a temporary copy of the template file will be created, i.e `apprc.in.tmp`. The script will then replace all the xrdb variables with the corresponding values read from the xrdb database. Then the script will overwrite the target file (`apprc`) with this temporary file. The template will remain untouched. 

NOTE: When you want to edit your config file, edit the template instead, otherwise the changes will be overwritten the next time you run the script. 

### Reload script
If the `-r` (or `-R`) flag is passed, the script will attempt to run a user-specified reload script located in the same directory as the target file. This script can be anything, but I suggest using it to restart applications that needs to be restarted before new changes to their config files can take affect.

Here's an example of a reload script I'm using for dunst: 

```sh
#!/bin/bash
# ~/.config/dunst/reload.sh

killall dunst

notify-send -u low      "Testing low"
notify-send -u normal   "Testing normal"
notify-send -u critical "Testing critical"
```

Dunst is automatically started again whenever a notification is sent. I send three new notifications in the script in order to see how the new changes apply. 

### From file
You often want to apply xrdb changes to many config files at once, probably whenever there's a change in your xrdb database. For this purpose, you can define a settings file where you specify all your config files.

Calling the script using such a file can be done as follows:

```sh
xrdb-replace -f ~/.config/xrdb-replace/files
```

Each line in this file should follow the same structure as the arguments you wish to pass to xrdb-replace. I.e

```sh
# ~/config/xrdb-replace/files
$XDG_CONFIG_HOME/rofi/colors.rasi  rofi 
$XDG_CONFIG_HOME/dunst/dunstrc     dunst -R
$XDG_CONFIG_HOME/zathura/zathurarc zathura -d "#"
```

For each line, xrdb-replace will be called with that line as its arguments. It's important to note that the arguments defined in the line read from the file takes presedence over the arguments initially passed to the script. E.g, if the script is called as follows, 

```sh
xrdb-replace -d "!" -f ~/.config/xrdb-replace/files
```

then `!` will be used as delimiter for all files except the zathurarc, where `#` will still be used.

I recommend creating a settings file and putting all the files you'd want to apply the script to there. I then suggest setting up an easy way of calling the script every time you reload xrdb. If you are using vim, an autocommand can be used for this purpose:

```sh
autocmd BufWritePost *Xresources,*Xdefaults !xrdb %; xrdb-replace -g -f $HOME/.config/xrdb-replace/files  
```

Place this in your vimrc, and every time you write to your Xresources or Xdefaults file, xrdb will be reloaded and xrdb-replace will be called with your settings file.

<!-- CONTRIBUTING -->
## Contributing
Suggestions are much appreciated. Make a fork and create a pull request, or message me directly. I'm still learning bash, so there's likely many things about the script that could be drasticly improved. 

<!-- LICENSE -->
## License
Distributed under the MIT License. See `LICENSE` for more information.

<!-- CONTACT -->
## Contact and social media
:mailbox_with_mail: [Email](mailto:anton@exlex.se) (the most reliable way of reaching me)

:camera: [Instagram](https://www.instagram.com/palmdrop/) (where I showcase some of my artistic projects)

:computer: [Blog](https://palmdrop.github.io/) (where I occasionally write posts about generative art)

<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements
Some of this code is based on a [comment](https://www.reddit.com/r/unixporn/comments/8giij5/guide_defining_program_colors_through_xresources/e1acuo0?utm_source=share&utm_medium=web2x&context=3) by reddit user [KD2NYT](https://www.reddit.com/user/KD2NYT/). 
