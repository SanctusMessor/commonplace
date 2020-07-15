# Creating Additional WSL2 Instances

Get a rootfs.   
e.g. Ubuntu rootfs: [https://wiki.ubuntu.com/WSL\#Installing\_Ubuntu\_on\_WSL\_via\_rootfs](https://wiki.ubuntu.com/WSL#Installing_Ubuntu_on_WSL_via_rootfs)

wsl.conf settings - [https://docs.microsoft.com/en-us/windows/wsl/wsl-config\#configure-per-distro-launch-settings-with-wslconf](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#configure-per-distro-launch-settings-with-wslconf)

Configure global options with .wslconfig - [https://docs.microsoft.com/en-us/windows/wsl/wsl-config\#configure-global-options-with-wslconfig](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#configure-global-options-with-wslconfig)

## Creating a Custom Distribution <a id="creating-a-custom-distribution"></a>

The first thing we’ll need is a root filesystem. Luckily Ubuntu make their WSL root filesystem available for download, which is available [here](https://cloud-images.ubuntu.com/releases/focal/release/ubuntu-20.04-server-cloudimg-amd64-wsl.rootfs.tar.gz). For this walk-through I’ve created a directory on my Windows C: drive called “wsl”, we’ll place the rootfs files in `c:\wsl\wslrootfs` and the distros in `c:\wsl\wsldistros\`

* Download the file above to the wslrootfs directory.
* Run `wsl.exe --import base-ubuntu C:\wsl\wsldistros\base-ubuntu\ C:\wsl\wslrootfs\ubuntu-20.04-server-cloudimg-amd64-wsl.rootfs.tar.gz`

In this command `base-ubuntu` Is just a name you want to assign so for example a specific project. `c:\wsl\wsldistros\base-ubuntu\` is the directory on your machine you want to place the virtual disk file for the distribution and then we have the distro file we downloaded

At this point you have a clean install of ubuntu 20.04 to use. You can then easily access each distribution you have available with Windows Terminal which places them all on a tab drop-down for easy access.

## Configure the base

Configure it however you please. Consider using something like [Ansible](https://docs.ansible.com/ansible/latest/user_guide/quickstart.html) if this is going to be a fairly common process for you.

At this point, I'm specifying my naming convention moving forward.  
`<date-modified>-<type>-<distro>`

* 20200714-base-ubuntu

## Using our WSL distro as a template <a id="using-our-wsl-distro-as-a-template"></a>

Once we’ve got the tooling we want installed, we can export the rootfs for later use.

```text
wsl --export baseubu c:\wsl\wslrootfs\20200714-base-ubuntu
```

Then we can create new instances based off this by importing the `20200714-base-ubuntu` file we just created

```text
wsl --import ViciousUnicorn c:\wsl\wsldistros\ViciousUnicorn\ c:\wsl\wslrootfs\20200714-base-ubuntu
```

and when we start it up, all our tools are in place :\)

## Cleaning up <a id="cleaning-up"></a>

Once you’re finished with it, it can just be removed with `wsl.exe --unregister <Name>`. This will delete the virtual disk file, however, as of writing the created folder is left behind.









### Acknowledgement

Credit to Rory: [https://raesene.github.io/blog/2020/05/31/Custom\_Pentest\_Distributions\_With\_WSL2/](https://raesene.github.io/blog/2020/05/31/Custom_Pentest_Distributions_With_WSL2/)

