# Raspberry Pi GUI



## Install `xorg` and `xinit`

```bash
sudo apt-get install --no-install-recommends xserver-xorg xinit
```



## Install RPD

```bash
sudo apt-get install raspberrypi-ui-mods
```

```bash
sudo apt-get install --no-install-recommends raspberrypi-ui-mods lxsession
```



Optional installs:

```bash
pi-greeter : The Raspberry Pi LightDM login theme
rpd-icons : The Raspberry Pi Desktop icon theme
gtk2-engines-clearlookspix : GTK Theme Engine (used to render Raspberry Pi LightDM login/desktop theme properly)
```



## Install LXDE

```bash
sudo apt-get install lxde-core lxappearance
sudo apt-get install lightdm
```

## Install XFCE

```bash
sudo apt-get install xfce4 xfce4-terminal
sudo apt-get install lightdm
```

## Install Mate

```bash
sudo apt-get install mate-desktop-environment-core
sudo apt-get install lightdm
```
