# Easy automatic setup
Please download the following package and watch this video : https://www.facebook.com/plugins/video.php?href=https%3A%2F%2Fwww.facebook.com%2Faudioinjector%2Fvideos%2F1753758851602240%2F :
audioinjector.octo.setup_0.3_all.deb.tar.gz

Once you have downloaded and extracted this .deb file (default extraction location is Downloads), type the following command to install the sound card :
[code]sudo dpkg -i Downloads/audioinjector.octo.setup_0.3_all.deb[/code]
You will have to run this command from the terminal to make it work, because it requires user input to download the official latest Pi firmware.

One final step - it is strongly recommended to remove pulseaudio (you can always install it again later if you need it) :
[code]sudo apt remove pulseaudio
[/code]

At this point, just reboot as you normally would, I like to do it from the command line sometimes :
[code]sudo reboot[/code]

Thats it - you should be setup.

[b][size=150]Manual setup[/size][/b]
You don't need to do this if you have followed the "Easy audomatic setup" section.
[b][size=150]Before you setup the Octo[/size][/b]
Rasbian pixel's lxpanel doesn't play nice with ALSA, it has a nasty problem where it clobbers .asoundrc files as it sees fit ([url=https://github.com/raspberrypi-ui/lxpanel/issues/13]and they don't seem to want to fix this[/url] problem, despite [url=https://github.com/raspberrypi-ui/lxpanel/issues/14]various issues[/url]). To fix this we disable the volume plugin like so (for the pi user) :
[code]sed -i 's/\=volumealsa/\=REMOVEvolumealsa/' ~pi/.config/lxpanel/LXDE-pi/panels/panel[/code]

Further, pulseaudio seems to cause strange problems when interfacing ALSA directly, so we remove it - note you can install it again later if for whatever reason you think you need it :
[code]sudo apt remove pulseaudio[/code]

[b][size=150]Low level setup[/size][/b]
Cut and paste the following code to a terminal. The code will get the latest Pi Linux Kernel (which support the Octo) and will setup your /boot/config.txt file :
[code]echo updating the kernel
sudo rpi-update

echo check the device tree overlay is setup correctly in the file /boot/config.txt ...
sudo bash -c "sed -i \"s/^\s*dtparam=audio/#dtparam=audio/\" /boot/config.txt"
cnt=`grep -c audioinjector-wm8731-audio /boot/config.txt`
if [ "$cnt" -eq "0" ]; then
        sudo bash -c "echo '# enable the AudioInjector.net sound card
dtoverlay=audioinjector-addons' >> /boot/config.txt"
fi
[/code]

After executing that script, you should be able to reboot and the Audio Injector Octo card should be working.

Your /boot/config.txt file should have these lines in it :
[code]#dtparam=audio=on
dtoverlay=audioinjector-addons
[/code]

If you wan to do this manually, then first update your kernel :
[code]sudo rpi-update[/code]

Next comment out the PWM audio and add the audioinjector overlay in the file /boot/config.txt :
[code]#dtparam=audio=on
dtoverlay=audioinjector-addons
[/code]

[b][size=150]High level setup[/size][/b]
Now that your card is running, you will want to allow the sound card to play between 1 and 8 channels and also record between 1 and 6 channels. Why ? you may ask .. because say you have a stereo audio file, you may want to play this out without converting it to 8 channels. To do this we need to create a file called the .asoundrc.

Save the following text to the file called '.asoundrc' in your home directory. If you don't know how to do that, the following is the file name to "save as"
[code]/etc/asound.conf[/code]
This is the content of that file :
[code]pcm.!default {
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
[/code]

Finally, also ensure that your .asoundrc is using the anyChannelCount plugin, paste the following to the file ~/.asoundrc :
[code]pcm.!default {
#       type hw
#       card 0
        type plug
        slave.pcm "anyChannelCount"
}

ctl.!default {
        type hw
        card 0
}[/code]

If you are having troubles setting up, then please post a new topic to this forum for help.
