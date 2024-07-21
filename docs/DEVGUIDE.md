# Dev Guide For The Jellyfin Roku App

Follow the steps below to install the app on your personal Roku device. This will enable you to write code for the app, install the latest beta release, as well as provide app logs to the developers if you encounter a bug.

- [Dev Guide For The Jellyfin Roku App](#dev-guide-for-the-jellyfin-roku-app)
  - [Developer Mode](#developer-mode)
  - [Clone the GitHub Repo](#clone-the-github-repo)
  - [Install Dependencies](#install-dependencies)
  - [Method 1: Visual Studio Code](#method-1-visual-studio-code)
    - [Install VSCode](#install-vscode)
    - [Usage](#usage)
    - [Hardcoding Roku Information](#hardcoding-roku-information)
  - [Method 2: Command Line](#method-2-command-line)
    - [Workflow](#workflow)
    - [Install Command Line Dependencies](#install-command-line-dependencies)
    - [Deploy](#deploy)
    - [Bug/Crash Reports](#bugcrash-reports)
    - [(Optional) Update Images](#optional-update-images)
  - [Committing](#committing)
  - [Bug Fixes](#bug-fixes)
  - [Adding a User Setting](#adding-a-user-setting)
    - [The order of any particular menu is as follows](#the-order-of-any-particular-menu-is-as-follows)
    - [When giving your setting a name](#when-giving-your-setting-a-name)
    - [When giving your setting a description](#when-giving-your-setting-a-description)
      - [**Remember to add all new strings to locale/en\_US/translations.ts**](#remember-to-add-all-new-strings-to-localeen_ustranslationsts)

## Developer Mode

Put your Roku device in [developer mode](https://blog.roku.com/developer/2016/02/04/developer-setup-guide). Write down your Roku device IP and the password you created - you will need these!

## Clone the GitHub Repo

Navigate to where you'd like to install the app then copy the application files:

```bash
git clone https://github.com/jellyfin/jellyfin-roku.git
```

Open up the new folder:

```bash
cd jellyfin-roku
```

## Install Dependencies

You'll need [`npm`](https://nodejs.org), version 16 at least.

Then, use it to install dependencies

```bash
npm install
```

## Method 1: Visual Studio Code

We recommend using Visual Studio Code when working on this project. The [BrightScript Language extension](https://marketplace.visualstudio.com/items?itemName=RokuCommunity.brightscript) provides a rich debugging experience, including in-editor syntax checking, debugging/breakpoint support, variable inspection at runtime, auto-formatting, an integrated remote control mode, and [much more](https://rokucommunity.github.io/vscode-brightscript-language/features.html).

### Install VSCode

1. Download and install [Visual Studio Code](https://code.visualstudio.com/)
2. Install the **BrightScript Language** extension within VSCode in the _Extensions_ panel or by downloading it from the [VSCode Marketplace](https://marketplace.visualstudio.com/items?itemName=RokuCommunity.brightscript).

### Usage

1. Open the `jellyfin-roku` folder in VSCode
2. Press `F5` on your keyboard or click `Run` -> `Start Debugging` from the VSCode menu. ![image](https://user-images.githubusercontent.com/2544493/170696233-8ba49bf4-bebb-4655-88f3-ac45150dda02.png)

3. Enter your Roku IP address and developer password when prompted

That's it! VSCode will auto-package the project, sideload it to the specified device, and the channel is up and running. (assuming you remembered to put your device in [developer mode](#developer-mode))

### Hardcoding Roku Information

Out of the box, the BrightScript extension will prompt you to pick a Roku device (from devices found on your local network) and enter a password on every launch. If you'd prefer to hardcode this information rather than entering it every time, you can set these values in your VSCode user settings:

```json
{
  "brightscript.debug.host": "YOUR_ROKU_HOST_HERE",
  "brightscript.debug.password": "YOUR_ROKU_DEV_PASSWORD_HERE"
}
```

Example:
![image](https://user-images.githubusercontent.com/2544493/170485209-0dbe6787-8026-47e7-9095-1df96cda8a0a.png)

## Method 2: Command Line

### Workflow

Modify code -> `make build-dev install` -> Use Roku remote to test changes -> `telnet ${ROKU_DEV_TARGET} 8085` -> `CTRL + ]` -> `quit + ENTER`

You will need to use telnet to see log statements, warnings, and error reports. You won't always need to telnet into your device but the workflow above is typical when you are new to BrightScript or are working on tricky code.

### Install Command Line Dependencies

You'll need [`make`](https://www.gnu.org/software/make) and [`curl`](https://curl.se).

Build the package

```bash
make build-dev
```

This will create a zip in `out/jellyfin-roku.zip`, that you can upload on your Roku's device via your browser.
Or you can continue with the next steps to do it via the command line.

### Deploy

Run this command - replacing the IP and password with your Roku device IP and dev password from the first step:

```bash
export ROKU_DEV_TARGET=192.168.1.234
export ROKU_DEV_PASSWORD=password
```

Package up the application, send it to your Roku, and launch the channel:

```bash
make install
```

Note: You only have to run this command once if you are not a developer. The Jellyfin channel will still be installed after rebooting your Roku device.

### Bug/Crash Reports

Did the app crash? Find a nasty bug? Use this command to view the error log and [report it to the developers](https://github.com/jellyfin/jellyfin-roku/issues):

```bash
telnet ${ROKU_DEV_TARGET} 8085
```

To exit telnet: `CTRL + ]` and then type `quit + ENTER`

You can also take a screenshot of the app to augment the bug report.

```bash
make screenshot
```

### (Optional) Update Images

This repo already contains all necessary images for the app. This script only needs to be run when the [official Jellyfin images](https://github.com/jellyfin/jellyfin-ux) are changed to allow us to update the repo images.

You'll need `convert`, from [ImageMagick](https://imagemagick.org)

Download and convert images:

```bash
make get_images
```

## Committing

Before committing your code, please run:

```bash
npm run lint
```

And fix any encountered issue.

## Bug Fixes

All Pull Requests that fix a bug in production should target the "bugfix" branch i.e. `2.0.z`, `2.1.z`, etc. All other Pull Requests should target the `master` branch.

## Adding a User Setting

Your new functionality may need a setting to configure its behavior, or, sometimes, we may ask you to add a setting for your new functionality, so that users may enable or disable it. If you find yourself in this position, please observe the following considerations when adding your new user setting.

### The order of any particular menu is as follows

1. Any menu titled "General."
2. Any settings that have children, in alphabetical order.
3. Any settings that do not have children, in alphabetical order.

### When giving your setting a name

Ideally, your setting will be named with a relevant noun such as `Cinema Mode` or `Codec Support.` Sometimes there is no such name that is sufficiently specific, such as with `Clock`. In this case you must use a verb phrase to name your setting, such as `Hide Clock.` If your verb phrase _must_ be long to be specific, you may drop implied verbs if absolutely necessary, such as how `Text Subtitles Only` drops the implied `Show.` Do not use the infinitive form `action-doing` or `doing stuff.` Instead, use the imperative: `Do Action` or `Do Stuff.` Remember that _characters are a commodity in names._

Generally, we should not repeat the name of a setting's parent in the setting's name. Being a child of that parent implies that the settings are related to it.

### When giving your setting a description

A setting's description should begin with a grammatically correct, complete, imperative sentence that ends with a period. _Characters are not a commodity in descriptions_ so be specific. Again, do not use infinitive verb phrases ("...ing" should not appear anywhere in the text of your setting). While the first sentence should be imperative, additional sentences may be necessary to tell your user how to use the setting or why its doing what its doing. If you _must_ use non-imperative sentences, be concise and consider the fact that your description will need to be translated into many languages. Do not use colloquialism, metaphor, or idiomatic phrases.

#### **Remember to add all new strings to locale/en_US/translations.ts**
