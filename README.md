# Easy automatic setup
Please download the following package and [watch this video](https://www.facebook.com/plugins/video.php?href=https%3A%2F%2Fwww.facebook.com%2Faudioinjector%2Fvideos%2F1753758851602240%2F) :
https://github.com/Audio-Injector/Octo/blob/master/audioinjector.octo.setup_0.3_all.deb

NOTE : no need to extract the deb file any more, simply install it.

Once you have downloaded the .deb file, type the following command to install the sound card :
```
sudo dpkg -i Downloads/audioinjector.octo.setup_0.3_all.deb
```
You will have to run this command from the terminal to make it work, because it requires user input to download the official latest Pi firmware.

One final step - it is strongly recommended to remove pulseaudio (you can always install it again later if you need it) :
```
sudo apt remove pulseaudio
```

At this point, just reboot as you normally would, I like to do it from the command line sometimes :
```
sudo reboot
```

Thats it - you should be setup.

# Manual setup
You don't need to do this if you have followed the "Easy audomatic setup" section.
# Before you setup the Octo
Rasbian pixel's lxpanel doesn't play nice with ALSA, it has a nasty problem where it clobbers .asoundrc files as it sees fit ([and they don't seem to want to fix this](https://github.com/raspberrypi-ui/lxpanel/issues/13) problem, despite [various issues](https://github.com/raspberrypi-ui/lxpanel/issues/14)). To fix this we disable the volume plugin like so (for the pi user) :
```
sed -i 's/\=volumealsa/\=REMOVEvolumealsa/' ~pi/.config/lxpanel/LXDE-pi/panels/panel
```

Further, pulseaudio seems to cause strange problems when interfacing ALSA directly, so we remove it - note you can install it again later if for whatever reason you think you need it :
```
sudo apt remove pulseaudio
```

# Low level setup
Cut and paste the following code to a terminal. The code will get the latest Pi Linux Kernel (which support the Octo) and will setup your /boot/config.txt file :
```
echo updating the kernel
sudo rpi-update

echo check the device tree overlay is setup correctly in the file /boot/config.txt ...
sudo bash -c "sed -i \"s/^\s*dtparam=audio/#dtparam=audio/\" /boot/config.txt"
cnt=`grep -c audioinjector-wm8731-audio /boot/config.txt`
if [ "$cnt" -eq "0" ]; then
        sudo bash -c "echo '# enable the AudioInjector.net sound card
dtoverlay=audioinjector-addons' >> /boot/config.txt"
fi
```

After executing that script, you should be able to reboot and the Audio Injector Octo card should be working.

Your /boot/config.txt file should have these lines in it :
```
#dtparam=audio=on
dtoverlay=audioinjector-addons
```

If you wan to do this manually, then first update your kernel :
```
sudo rpi-update
```

Next comment out the PWM audio and add the audioinjector overlay in the file /boot/config.txt :
```
#dtparam=audio=on
dtoverlay=audioinjector-addons
```

# High level setup
Now that your card is running, you will want to allow the sound card to play between 1 and 8 channels and also record between 1 and 6 channels. Why ? you may ask .. because say you have a stereo audio file, you may want to play this out without converting it to 8 channels. To do this we need to create a file called the .asoundrc.

Save the following text to the file called '.asoundrc' in your home directory. If you don't know how to do that, the following is the file name to "save as"
```
/etc/asound.conf
```
This is the content of that file :
```
pcm.!default {
#       type hw
#       card 0
        type plug
        slave.pcm "anyChannelCount"
}

ctl.!default {
        type hw
        card 0
}

pcm.anyChannelCount {
    type route
    slave.pcm "hw:0"
    slave.channels 8;
    ttable {
           0.0 1
           1.1 1
           2.2 1
           3.3 1
           4.4 1
           5.5 1
           6.6 1
           7.7 1
    }
}

ctl.anyChannelCount {
    type hw;
    card 0;
}
```

Finally, also ensure that your .asoundrc is using the anyChannelCount plugin, paste the following to the file ~/.asoundrc :
```
pcm.!default {
#       type hw
#       card 0
        type plug
        slave.pcm "anyChannelCount"
}

ctl.!default {
        type hw
        card 0
}
```

If you are having troubles setting up, then please post to a relevant issue to get help.
