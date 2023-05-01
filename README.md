# archupdater

Arch updater is a script to somewhat simplify and automate system upgrade for Arch Linux

## Prerequisites

Arch Linux installation with `core` package group. Configuration file `archupdater.conf` has to be present in the same directore as `archupdater` script itself; copy `archupdater.conf.sample` and modify to your needs.

## How it works

`archupdater` has to be run with root permissions. It first parses configuration file and does a sanity check on input. Then it force-refreshes all package databases by running `pacman -Syy`. User will be prompted before refresh happens.

Now it gathers the list of upgradable (out-of-date) packages by running `pacman -Qu` and filters out packages listed in `ARCHUPDATER_PACKAGE_IGNORE_LIST`; this comes in handy later when it determines what (if any) post-upgrade command to run.

At this point, it has all necessary information to put together upgrade and post-upgrade commands.

Upgrade command will take the form of `pacman -Su --ignore=pkg1,pkg2,pkg3,... --ignoregroup=pkg1,pkg2,pkg3,...` where `--ignore` and `--ignoregroup` are, respectively, ommitted completely if `ARCHUPDATER_PACKAGE_IGNORE_LIST` and `ARCHUPDATER_PACKAGE_IGNOREGROUP_LIST` are not present or are empty.

Post-upgrade command is determined in the following, cascading fashion:
* if any package in `ARCHUPDATER_BOOTLOADER_INSTALL_TRIGGER_PACKAGE_LIST` appears in filtered upgradable package list, the command will take the form of `$ARCHUPDATER_BOOTLOADER_INSTALL_COMMAND && $ARCHUPDATER_BOOTLOADER_UPDATE_COMMAND`; as an example from the sample configuration file, if `grub` or `efibootmgr` packages are about to be upgraded, the command will look like `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB && grub-mkconfig -o /boot/grub/grub.cfg`
* if any Linux kernel package (`linux[[:alnum:]-]*`) appears in filtered upgradable package list, the command will take the form of `$ARCHUPDATER_BOOTLOADER_UPDATE_COMMAND`; as an example from the sample configuration file, the command will look like `grub-mkconfig -o /boot/grub/grub.cfg`
* if neither of the previous cases are valid, post-upgrade command will be empty

Now, everything is ready to run upgrade and post-upgrade commands. The script prints out its command plan and prompts user before procedure begins. Note that `pacman -Su ...` still requires user to confirm upgrade, at which point `CTRL+C` can still be invoked to cancel the entire upgrade and nothing will be changed apart from previously (force-)refreshed package databases.

## Suggestions

As the author of this script and, although new to Arch, nearly two decade veteran of Linux (initially Slackware and now Void), I strongly suggest every Arch (or otherwise) user to have at least two kernels installed on their systems; in the case of Arch, I suggest `linux` and `linux-lts` packages. Further, I would suggest that `linux-lts` is kept permanently on ignore list and is only upgraded by hand, if at all. It is a royal pain in the posterior dealing with a non-booting machine.

Also note that if you plan to put any kernel package on ignore list, I strongly suggest, if applicable to your case, putting all packages relating to that kernel package on ignore list as well (like headers or DKMS packages). In my particular case, I have `linux-lts`, `linux-lts-headers`, and `nvidia-lts` packages on ignore list, so in case `linux` does not boot, I can always reboot in `linux-lts` and have fully working system with GUI to troubleshoot the problem.
