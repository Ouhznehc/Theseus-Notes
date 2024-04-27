# Get Started

Since I use macOS Sonoma Version 14.3.1(Apple M1 Pro) for study, only guidance on **MacOS** will be introduced below. As for other systems, check out (here)[https://github.com/theseus-os/Theseus] for more information.

## Setting up the environment

1. Obtain the Theseus repository (with all submodules): 
```sh
git clone --recurse-submodules --depth 1 https://github.com/theseus-os/Theseus.git
```
2. Install Rust: 
```sh
curl https://sh.rustup.rs -sSf | sh
```
**NOTE**: on MacOS, you need to run `gmake` instead of `make` for build commands (or you can simply create a shell alias). This is because HomeBrew installs its binaries in a way that doesn't conflict with built-in versions of system utilities.

3. Create a shell alias:
```sh
# edit your `.zshrc` or `.bashrc`
alias make="gmake"
```
4. Install [HomeBrew](https://brew.sh/), then run the MacOS build setup script:
```sh
sh ./scripts/mac_os_build_setup.sh
```
If things go wrong, remove the following build directories and try to run the script again.
```sh
rm -rf /tmp/theseus_tools_src
```


## IDE setup

My personal preference is to use [VS Code](https://code.visualstudio.com/), which has excellent cross-platform support for Rust. Other options are available [here](https://areweideyet.com/).
For VS Code, recommended plugins are:
- **rust-analyzer**, by matklad
- **Better TOML**, by bungcip
- **x86 and x86_64 Assembly**, by 13xforever

## Other

The official repo provide other tools like **Debugging Theseus on QEMU** or **Booting on Real Hardware**. Feel free to explore more!

