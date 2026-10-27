# ST Cloud Fixer

""ST Cloud"" for 'lua' games.

**This the only official source for ST Cloud Fixer. Any other websites are not operated by me or are otherwise endorsed by this project. At least one of those fake sites was actively distributing malware!**

> ****This software is experimental and under active development.**** The underlying techniques are fairly insane. What this software tries to do is nuts to attempt. This software could damage your save files and probably will! It could overwrite your saves, cause weird conflicts, make your saves disappear, make you cry. Back up any saves you care about before using this software.

But it probably won't. It's been very solid for a long time.

>****Do not use this if you do not actively want cloud saves for "lua" games. If all you care about is the ST Cloud error, disable ST Cloud in properties for that game.****

## What it does

First, some back story:

Once upon a time, there was a tool called STTools. STTools supported cloud saving, kind of.

Later, Valve patched the (sinful) thing STTools did to sync saves. Specifically, STTools rewrote requests to AppID 760, which is ST Screenshots. It sent all ST Cloud requests for non-owned AppIDs there. It did not create prefixes for each individual game, which means that each lua app shared the saves with all others. This can cause saves to conflict if multiple games use the same save file name. This also means that your saves are replicated in the `ST/Userdata/<STid>/<appid for lua game>` folder for each lua app.

It also did not support ST AutoCloud games at all. It would simply show a fake success message for those games.

What _this_ tool does is redirect ST Cloud requests for games that are injected to Google Drive/OneDrive/a local folder, including AutoCloud games. Everything is native inside the ST Client, but the actual data is read/written to and from your cloud account. This was much harder to do than just redirecting read/write to an AppID that your account owns, but it was fun to make. It also is less likely to piss off Valve.

## How it works

ST Cloud Fixer for Windows consists of a C++ DLL and a WPF companion app:

1. The companion app is used to copy the DLL to your ST folder and let the user login to their cloud provider.
2. The DLL hooks ST's internal cloud save RPC handlers via ~~vtable interception~~ black magic.
3. When a lua game attempts to read or write cloud save data, the DLL intercepts the calls and redirects it. If the game is owned, the game uses normal ST Cloud as expected. If a lua is present that only unlocks DLC, the game will use normal ST Cloud.
4. More dark magic occurs. Saves sync. Bytes flow. This all is visible in the ST UI and looks identical to normal ST Cloud functionality.

Same rough idea on Linux, but involving a flatpak application and a library that is loaded on ST startup instead. 
   
## Supported cloud providers

- **Google Drive**
- **OneDrive**
- **Cloudflare R2**
- **S3-Compatible** - AWS S3, MinIO, Backblaze B2, Wasabi, DigitalOcean Spaces, and any self-hosted S3 server.
- **Local folder / mapped drive** - by request of literally one user.

With more to come over time. 

## Usage (Windows)

Grab the latest release from the [Releases page](https://github.com/martinj50/STCloudFixer/releases).

Run the EXE. Pick your mode - STfixer mode for fixes to ST bugs, ST Cloud Fixer mode for the good stuff. In Setup, hit 'Run All Patches'. Go to the Cloud Provider tab, select your provider. If it is a cloud provider, sign in to it.

That's it. Go launch ST and watch the magic.

## Usage (Linux)

Edit your SLSST config, set DisableCloud to No. 


```curl -fsSL headcrab.pages.dev | bash```

Open the ST Cloud Fixer app, sign into a provider.

Edit your SLS config. The games you want to sync must be specified under AdditionalApps in your SLS config. This requirement will go away in the future. 

Now launch ST and watch your games sync!

## Building from source (Windows)

### Prerequisites

- Visual Studio 2022 (or Build Tools) with the C++ and .NET 8 workloads
- CMake 3.20+

### Build

```bash
cmake -B build -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release
```

This builds both the C++ DLL (`build/Release/cloud_redirect.dll`) and publishes the WPF app (`ui/bin/publish/ST Cloud Fixer.exe`). The DLL is automatically embedded into the executable.

Or don't build it? Building Windows apps is pain.

## Building from source (Linux)

Oooooh boy. Yeah. Have fun. 

You need to build against glibc 2.31 or older. Ubuntu 20.04 would work, if you dislike yourself. There's also an ancient version of Fedora that fits the bill. Or Debian 11. Distrobox is the way, here. Don't even bother trying to build under whatever distro you daily, you'll wind up fighting it for no reason. Distrobox exists for a reason. 

If you are building under Ubuntu 20.04, GCC 12 is needed along with the 32-bit multilib stuff. System cmake is ancient garbage, you'll have to update it.

Then specify -DLINUX_32BIT=ON and wham bam.

Isn't building for weird distros _fun?_

## Contibuting

ST Cloud Fixer is open source. I welcome PRs. I love PRs.

I don't _love_ downstream forks. You have a right to fork and do what you want, but if you make something useful, if you fix issues...I'd much prefer they be sent upstream! Help me help you! If you identify a problem/add a provider/improve the tool in any way, please consider submitting your changes as a PR, rather than keeping them in a downstream fork.
