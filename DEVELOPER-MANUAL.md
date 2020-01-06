## visualboyadvance-m Developer Manual

Here we will keep notes about our development process, policies and environment
setup guides.

### Windows Native Development Environment Setup

Install Visual Studio 2019:

You just want the core and C++ packages.

Install the chocolatey package manager:

- Press Win+X and open Windows PowerShell (administrator).

- Run this command:

```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

Close the administrator PowerShell window and open it again.

Install some chocolatey packages:

```
choco install ag dejavufonts git gpg4win hackfont microsoft-windows-terminal poshgit powershell-preview vim-tux zip unzip notepadplusplus openssh
```

Launch the terminal and choose Settings from the tab dropdown, this will open
the settings json in visual studio.

Add the Tango Dark Theme to the theme section:

```
    {
      "background": "#000000",
      "black": "#000000",
      "blue": "#3465a4",
      "brightBlack": "#555753",
      "brightBlue": "#729fcf",
      "brightCyan": "#34e2e2",
      "brightGreen": "#8ae234",
      "brightPurple": "#ad7fa8",
      "brightRed": "#ef2929",
      "brightWhite": "#eeeeec",
      "brightYellow": "#fce94f",
      "cyan": "#06989a",
      "foreground": "#d3d7cf",
      "green": "#4e9a06",
      "name": "Tango Dark",
      "purple": "#75507b",
      "red": "#cc0000",
      "white": "#d3d7cf",
      "yellow": "#c4a000"
    }
```

Change the powershell profile like so:

```
      {
        "acrylicOpacity": 0.5,
        "closeOnExit": true,
        "colorScheme": "Tango Dark",
        "commandline": "C:\\Program Files\\PowerShell\\7-preview\\pwsh.exe",
        "cursorColor": "#FFFFFF",
        "cursorShape": "filledBox",
        "fontFace": "Hack",
        "fontSize": 10,
        "guid": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
        "historySize": 30001,
        "icon": "ms-appx:///ProfileIcons/{574e775e-4f2a-5b96-ac1e-a2962a402336}.png",
        "backgroundImage": "ms-appdata:///roaming/wallhaven-127481.jpg",
        "backgroundImageOpacity": 0.32,
        "backgroundImageStretchMode": "uniformToFill",
        "name": "PowerShell Core",
        "padding": "0, 0, 0, 0",
        "snapOnInput": true,
        "startingDirectory": "%USERPROFILE%",
        "useAcrylic": false
      },
```

Notice I have a background image, if you want to install one, follow this
guide:

https://www.thomasmaurer.ch/2019/09/how-to-change-the-windows-terminal-background-image/

Change the settings for the tab shortcuts if you want, I use the same config as
kitty:

```
            {
                "command" : "newTab",
                "keys" : 
                [
                    "ctrl+shift+t"
                ]
            },
            {
                "command" : "nextTab",
                "keys" : 
                [
                    "ctrl+shift+right"
                ]
            },
            {
                "command" : "prevTab",
                "keys" : 
                [
                    "ctrl+shift+left"
                ]
            },
```

This works kind of like a tmux for powershell!

Now add some useful things to your powershell profile:

Run:

```
notepad++ $profile
```

Here's mine, most importantly the Visual Studio environment setup:

```
chcp 65001 > $null

Set-Culture en-US

$env:EDITOR        = 'C:/Program\ Files/Vim/vim81/vim.exe'
$env:VBAM_NO_PAUSE = 1

function Get-ChildItem-MegaBytes {
    ls $args | Select-Object Name, @{Name="MegaBytes";Expression={$_.Length / 1MB}}
}

function Get-CMake-Configuration {
    ag 'CMAKE_BUILD_TYPE|VCPKG_TARGET_TRIPLET' CMakeCache.txt
}

Set-Alias -name megs    -val Get-ChildItem-MegaBytes
Set-Alias -name notepad -val "C:\Program Files\Notepad++\notepad++.exe"
Set-Alias -name cmconf  -val Get-CMake-Configuration
Set-Alias -name which   -val Get-Command
Set-Alias -name grep    -val ag

# only load VS env once
if (-not $env:VSCMD_VER) {
    pushd "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build"
        cmd /c "vcvars64.bat&set" |
        foreach {
            if ($_ -match "=") {
                $v = $_.split("="); set-item -force -path "ENV:\$($v[0])"  -value "$($v[1])"
            }
        }
    popd
}

# Chocolatey profile
$ChocolateyProfile = "$env:ChocolateyInstall\helpers\chocolateyProfile.psm1"
if (Test-Path($ChocolateyProfile)) {
    Import-Module "$ChocolateyProfile"
}

Import-Module 'C:\tools\poshgit\dahlbyk-posh-git-9bda399\src\posh-git.psd1'

Import-Module PSReadLine

Set-PSReadLineOption -EditMode Emacs
```

You may want to add an alias for your favorite editor.

Relaunch the terminal.

To set notepad++ as the git commit editor, run this command:

```
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession"
```

To configure truecolor support for vim, add this to your `~/.vimrc`:

```
if !has('gui-running')
  set bg=dark
  set termguicolors
endif
```

To use 256 color support instead, use this:

```
if !has('gui-running')
  set bg=dark
  set t_Co=256
endif
```

I also recommend adding this to spellcheck commit messages:

```
au BufRead COMMIT_EDITMSG setlocal spell
```

The most important thing that should be in your `~/.vimrc` is of course:

```
filetype plugin indent on
```

If you don't know how to use vim and want to learn, run `vimtutor`, it takes
about half an hour.

Now to set up gpg.

I recommend not using a passphrase so as not to deal with passphrase prompts,
which are a huge pain and break both ssh and release automation. If you have a
passphrase and want to remove it see:

http://www.peterscheie.com/unix/automating_signing_with_GPG.html

Configure git to use gpg4win:

```
git config --global gpg.program "c:/Program Files (x86)/GnuPG/bin/gpg.exe"
```

Tell git to always sign commits:

```
git config --global commit.gpgsign true
```

Now your development/build environment is ready!

To set up ssh into your powershell environment, which allows doing builds
remotely etc., edit the registry as described here to set powershell-preview as
the default shell:

https://github.com/PowerShell/Win32-OpenSSH/wiki/DefaultShell

Follow this guide to set up the server:

https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH

### PowerShell Notes

PowerShell is very different from traditional UNIX shells, I am very new to it
myself, but I will pass on some tips here.

First, read this guide:

https://mathieubuisson.github.io/powershell-linux-bash/

You can use `ag` to both search and as a substitute for `grep`.

For example:

```
alias | ag sort
```

For `ls -ltr` use:

```
ls | sort lastwritetime
```

You can use the `-Recursive` flag for `Get-ChildItem` (`ls`) as a substitute
for `find`, e.g.:

```
ls -rec *.xrc
```

Now combine this with the awesome powershell object pipeline to e.g. delete all
vim undo files:

```
ls -rec .*un~ | rm
```

You will notice that `Remove-Item` (`rm`) does not take multiple space
separated items, you must separate them by commas, eg.:

```
1..4 | %{ni foo$_; ni bar$_}
rm foo*, bar*
```

The equivalent of `rm -rf` to delete a directory is:

```
rm -rec dir
```

force is unnecessary, but if it is:

```
rm -rec -for dir
```

It should not take you very long to learn enough basic usage for your dev
workflow.
