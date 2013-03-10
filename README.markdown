This is a fork of [liskin's screenclone][liskin-screenclone], adding support for multiple outputs

# hybrid-screenclone

This is a reimplementation of [hybrid-windump][hybrid-windump] with the
opposite use-case: doing all rendering using the integrated intel card and
using the additional card just to get more outputs (e.g. a triple-head with
ThinkPad T420). As such, it uses the DAMAGE extension to avoid unnecessary
redraws and the RECORD extension to capture mouse movements, which are
translated to mouse movements on the destination X server.

For this to work correctly, an additional virtual Xinerama screen must be
available. To get one, see my [virtual CRTC for intel][patch] patch.

## Multiple outputs
This version supports remapping to a target Xinerama screen other than 0, and also remapping multiple screens at once. This is mostly useful if you're using a version of [the virtual CRTC patch][multiple-virtual-crtc] new enough to support multiple virtual screens, e.g.

    Section "Device"
        Identifier     "integrated"
        Driver         "intel"
        Option         "Virtuals" "2"
    EndSection

Resulting in a setup with 3 outputs: LVDS1, VIRTUAL1, and VIRTUAL2. Then

    xrandr --output VIRTUAL1 --mode 1600x1200 --right-of LVDS1 \
           --output VIRTUAL2 --mode 1600x1200 --right-of VIRTUAL1
    screenclone -x 1:0 -x 2:1

# Bumblebee

If you are running an optimus setup with bumblebee, this fork also adds direct support for that. Simply add -b to your screenclone command, and remove -d :display. Screenclone will connect to bumblebee (launching the nVidia X server), and get the display name from it. When screenclone (and anything else using it) disconnects, bumblebee will close the server and power down the discrete graphics card.

Normally, bumblebee does not attach any screens when it launches the nVidia card. You will need to alter /etc/bumblebee/xorg.conf.nvidia (or whatever path you are using), to add a screen section and refer to it in ServerLayout.

    Section "ServerLayout"
        ...
        Screen      "Screen0"
    EndSection

    Section "Screen"
        Identifier     "Screen0"
        Device         "Device0"
        DefaultDepth    24
        SubSection     "Display"
        Depth       24
        EndSubSection
    EndSection


[hybrid-windump]: https://github.com/harp1n/hybrid-windump
[patch]: https://github.com/liskin/patches/blob/master/hacks/xserver-xorg-video-intel-2.18.0_virtual_crtc.patch
[liskin-screenclone]: https://github.com/liskin/hybrid-screenclone
[multiple-virtual-crtc]: https://github.com/liskin/patches/blob/master/hacks/xserver-xorg-video-intel-2.20.14_virtual_crtc.patch
