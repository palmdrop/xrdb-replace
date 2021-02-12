<!-- PROJECT LOGO -->
<br />
<p align="center">
  <h1 align="center">xrdb-replace</h1>

  <p align="center">
    A script for generating config files using templates and xrdb: perfect for pesky applications that refuse to allow configuration through Xresources.
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
    <li><a href="#usage">Usage</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact and social media</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## About The Project
Being able to customize application at a unified location is hugely benefit, especially when trying to achieve a coherent colorscheme. A great deal of applications support configuration using Xresources, but far from all. When using [dunst](), [alacritty]() or [zathura](), changing colors and other settings have to be changed using the individual config files of those applications, which means that you have to change your colors in mutliple places whenever you redesign your system. What a hazzle.

I got tired of this and created a script which automates the process. The script takes a template config file where xrdb variables can be used, and produces a complete config file using this.

The script also works with a config file which can contain all the files you have made templates for, and the settings required to convert these templates into a new version of the config file. 

Every time you edit Xresources or make changes in the template, run this script for your new changes to take effect.

Some applications, however, require restart for new settings to be used. The script also has support for running a reload script written by the user, which could be configured to restart the affected applications.

<!-- GETTING STARTED -->
## Getting Started
This is just a simple bash script. Getting started shouldn't be that complicated.

### Prerequisites
I can't imagine that any of these utilities are not already part of your system, but here's a short list:

* xrdb
* grep
* sort
* sed
* cut

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

Here's the help description:

[PRINT BASIC USAGE]

An example of basic usage:

```sh
xrdb-replace ~/.config/dunst/dunstrc dunst
```

This tells the script to replace the "dunstrc" with a template which should be named "dunstrc.in", using the xrdb keyword "dunst". This means that there should exist a template file "dunstrc.in" which is identical to your regular dunstrc file but with the exception for wherever you want to use a xrdb variable. E.g, "%dunst.background%" ("%" is the default deliminator) will be replaced with the corresponding value found in the xrdb database. 

When using the script with a settings file, this is a possible usage:

```sh
xrdb-replace -f ~/.config/xrdb-replace/files
```

Each line in this file should follow the same structure as the arguments you wish to pass to xrdb-replace. I.e

```
$XDG_CONFIG_HOME/rofi/colors.rasi  rofi 
$XDG_CONFIG_HOME/dunst/dunstrc     dunst -R
$XDG_CONFIG_HOME/zathura/zathurarc zathura -d "#"
```

For each line, xrdb-replace will be called with that line as its arguments. It's important to note that the arguments defined in the line read from the file takes presedence over the arguments initially passed to the script. E.g, if the script is called as follows, using the same settings file

```sh
xrdb-replace -d "!" -f ~/.config/xrdb-replace/files
```

then "!" will be used as delimiter for all files except the zathurarc, where "#" will still be used.

I recommend creating a settings file and putting all the files you'd want to apply the script to there in the described format. I then suggest setting up an easy way of calling the script every time you reload xrdb. 

If you are using vim/neovim, I recommend an auto command for this purpose. 

```sh
autocmd BufWritePost *Xresources,*Xdefaults !xrdb %; xrdb-replace -f -g $HOME/.config/xrdb-replace/files  
```

Place this in your vimrc, and ever time you write to your Xresources or Xdefaults file, xrdb will be reloaded and xrdb-replace will be called with your settings file.

<!-- CONTRIBUTING -->
## Contributing
Suggestions are much appreciated. Make a fork and create a pull request, or message me directly. I'm still learning bash, so there's likely many things about the script that could be drasticly improved. 

<!-- LICENSE -->
## License
Distributed under the MIT License. See `LICENSE` for more information.

<!-- CONTACT -->
## Contact and social media
:mailbox_with_mail: [Email](mailto:anton@exlex.se) (the most reliable way of reaching me)

:camera: [Instagram](https://www.instagram.com/palmdrop/) (where I showcase a lot of my work)

:computer: [Blog](https://palmdrop.github.io/) (where I occasionally write posts about my process)

<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements
Some of this code is based on a [comment](https://www.reddit.com/r/unixporn/comments/8giij5/guide_defining_program_colors_through_xresources/e1acuo0?utm_source=share&utm_medium=web2x&context=3) by reddit user [KD2NYT](https://www.reddit.com/user/KD2NYT/). 
