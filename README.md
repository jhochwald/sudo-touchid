# sudo-touchid
`sudo-touchid` is a fork of `sudo` with Touch ID support on macOS (powered by the `LocalAuthentication` framework). Once compiled, it will allow you to authenticate `sudo` commands with Touch ID in the Terminal on supported Macs (such as the late 2016 MacBook Pros).

## Why this fork?

I like the idea of `sudo-touchid`, but the author doesn't merge the [existing password Fallback patch](https://github.com/mattrajca/sudo-touchid/pull/15) from [serverwentdown](https://github.com/serverwentdown).
So I merged that patch and did some tests. It works very well on MacOS 10.12. Tested it on 12.12.5 (16F73) and 10.12.6 Beta (16G8c).

There are no further changes at the moment.

### Binary

You will find a build of `sudo` and `visudo` in the /Dist/bin Directory.
I (personaly) can't recommend to use a pre-builded version of a tool like `sudo` or `visudo`!
But that is just my opinion.

### Package

You will find a Packed Version `sudo-touchid.pkg` in the /Dist/pkg Directory.
The Package contains the latest Binary (See above) and it also applies the correct permission.
I use this package internal to deploy it via [Munki](https://github.com/munki/munki/wiki).

The installer itself needs you Admin credentials to install the latest `sudo` and `visudo` binaries to `/usr/local/bin` and to change the permission.

#### Installer

The Installer itself is nothing fancy! There are no real options, just a bery basic installer (See above why).

<img src="images/installer1.png?raw=true" />

Only the destination could be changed (if applicable)

<img src="images/installer2.png?raw=true" />

You need to enter the Admin credentials (You need Admin permission to install the package).

<img src="images/installer3.png?raw=true" />

## Screenshot

<img src="images/Screenshot.png?raw=true" width=556 height=284 />

### Fallback

If you choose "Use Password" in the dialog above, the Fallback will come up:

<img src="images/auto_fallback.png?raw=true" />

Handy, if the Touch ID is not working or the Lid is closed.

This will not work an a Mac without Touch-ID.

## Warning

- The author not a security expert. While he is using this as a fun experiment on his personal computer, your security needs may vary.
- This has only been tested on the 2016 15" MacBook Pro with Touch Bar running macOS 10.12.1. (By the Author)
- There where tests with a 2016 MacBook Pro Touch Bar running macOS 12.12.5 as well.
- I recommend not to use the pre builded Binary and/or Package. Again, I would not use them because i would like to review the code and build them for myself. But this is just my personal opinion.

## Building

To build `sudo-touchid`, simply open the included Xcode project file with Xcode 8+, select the `Build All` target, and click **Build**.

## Running

If we try running our newly-built `sudo` executable now, we'll get an error:

> sudo must be owned by uid 0 and have the setuid bit set

To fix this, we can use our system's `sudo` command and the `chown/chmod` commands to give our newly-built `sudo` the permissions it needs:

> cd (built-products-directory)

> sudo chown root:wheel sudo && sudo chmod 4755 sudo

Now if we try running our copy of `sudo`, it should work:

> cd (built-products-directory)

> ./sudo -s

If you don't have a Mac with a biometric sensor, `sudo-touchid` will ~~fail~~ do nothing!. If you'd still like to test whether the `LocalAuthentication` framework is working correctly, you can change the `kAuthPolicy` constant to `LAPolicyDeviceOwnerAuthentication` in `sudo/plugins/sudoers/auth/sudo_auth.m`. This will present a dialog box asking the user for his or her password:

This is where the Fix (Described at the Top) comes into the game:
<img src="images/auto_fallback.png?raw=true" />
The Fallback work fine now. Very usefull if the MacBook Pro lid is closed.

While not useful in practice, you can use this to verify that the `LocalAuthentication` code does in fact work.

### Workaround
Use the Systems build in Sudo: `/usr/bin/sudo`

## Installing using Homebrew

I didn't test this, and I have no idea if the patch is implemented here, or not. But I think not.

`Xcode` is not required (As long as *HOMEBREW_PREFIX* is default).

> brew tap paulche/sudo-touchid

> brew install sudo-touchid

Follow caveat message to change owner/mode.

## Installing from source code

Replacing the system's `sudo` program is quite risky (can prevent your Mac from booting) and requires disabling System Integrity Protection (aka "Rootless").

Instead of replacing `sudo`, we can install our build under `/usr/local/bin` and give the path precedence over `/usr/bin`, this way our build is found first.

> sudo cp (built-products-directory)/sudo /usr/local/bin/sudo

> sudo chown root:wheel /usr/local/bin/sudo && sudo chmod 4755 /usr/local/bin/sudo

You can set up your `PATH` by adding `export PATH=/usr/local/bin:$PATH` to `.bashrc` (thanks @edenzik).

Now you should be able to enter `sudo` in any Terminal (or iTerm) window and authenticate with Touch ID!
