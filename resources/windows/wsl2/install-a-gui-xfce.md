# Install a GUI \(xfce\)

#### Ubuntu GUI commands: 

* sudo apt update && sudo apt -y upgrade
* sudo apt-get purge xrdp
* sudo apt install -y xrdp
* sudo apt install -y xfce4
* sudo apt install -y xfce4-goodies
* sudo cp /etc/xrdp/xrdp.ini /etc/xrdp/xrdp.ini.bak
  * **Optional:** sudo sed -i 's/3389/3390/g' /etc/xrdp/xrdp.ini
* sudo sed -i 's/max\_bpp=32/\#max\_bpp=32\nmax\_bpp=128/g' /etc/xrdp/xrdp.ini
* sudo sed -i 's/xserverbpp=24/\#xserverbpp=24\nxserverbpp=128/g' /etc/xrdp/xrdp.ini
* echo xfce4-session &gt; ~/.xsession
* sudo vim /etc/xrdp/startwm.sh

comment these lines to:

```text
#test -x /etc/X11/Xsession && exec /etc/X11/Xsession
#exec /bin/sh /etc/X11/Xsession
```

add these lines:

```text
# xfce
startxfce4
```

* sudo /etc/init.d/xrdp start

