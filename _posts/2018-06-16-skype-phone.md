---
layout: "post"
title: "Skype Phone - Freetalk SP2014"
date: "2018-06-16 22:09"
---

I picked up a phone that was going to electronics recycling.

It is a Freetalk TALK-3000 or SP2014.  It's also rebranded as ECO XS2008CA and Belkin Desktop Internet Phone for Skype F1PP010EN-SK.

![skype-phone-front](/images/2018/06/skype-phone-front.png)

In around August 2017, users report suddenly having problems logging in with the phone [(link)](https://answers.microsoft.com/en-us/skype/forum/skype_other-skype_connect/belkin-desktop-phone-for-skype/bd7a7c7a-8630-46d0-8dc2-23fa0f22fbb0).  It appears that Skype, and likely the manufacturer, no longer supports this phone.

Initially, my goal was only to learn more about the device and scavenge it for parts.

### Serial console
I opened it up, looking for UART or serial port pins.

![skype-phone-pcb](/images/2018/06/skype-phone-pcb.png)

The connector J1 had 4 pins, and was a good candidate.  Ground was found with visual inspection and confirmed with a multimeter.  The other pins identified by trial and error with a USB-UART converter.

![skype-phone-serial](/images/2018/06/skype-phone-serial.png)

I used minicom, set to fairly standard settings of 115200-8-n-1, no flow control, and was able to see the device boot up.

Trimmed version of log:

```
s..
ssssssssssssssss
DPAPRWP
nand:8 bit
NAND ID=0xec75
psize 0x00000200
magic 0xbabeface
size  0x00008400
csum  0x000000dd
Jump 0x8001006c
nand:8 bit
NAND ID=0xec75

Broadcom Linux NAND boot v1.0

Press any key to enter download mode [....]
Skipping download mode
(...)
Linux version 2.6.17.14-BROADCOM (root@Ken) (gcc version 3.4.6) #12 PREEMPT Thu Sep 16 10:58:04 CST 2010
(...)

```

### Obtaining root

```
Please press Enter to activate this console.

BusyBox v1.01 (2009.08.03-09:32+0000) Built-in shell (ash)
Enter 'help' for a list of built-in commands.

#
# id
uid=0(root) gid=0(root)
```

Turns out the console runs at root already... I was not expecting it to be that easy.

### Repurposing the device

As far as I know, Skype no longer works on this device and is likely considered End of Life by the manufacturer.

Since we have root access already, it should be possible to use the device for other things.

Some ideas:
* Other VOIP or voice chat service with open source libraries
* generic SIP client  [(wiki)](https://en.wikipedia.org/wiki/Session_Initiation_Protocol)
* some kind of server (web, ...)
* run a game
* And many more possibilities, similar to routers and Raspberry Pi.

The best use for the device, given it's a phone, would be using it for VOIP or SIP.  But, I don't have any use for something like that.

A game would be a more fun project, and some of the work in getting it to run would carry-over to a VOIP client too.

### Plan

In order to run a game or VOIP client, I would need to:
- [ ] be able to compile and my own code
- [ ] draw to the screen
- [ ] play audio through the speaker or handset
- [ ] read the input keys

### Toolchain
Looking up the SoC online and at /proc/cpuinfo let me know I needed a MIPS cross-compiler .
There are ready-built compilers available online, but after more searching, I found Belkin has some source of the GPL parts available.

I downloaded F1PP010EN-SKv1_GPL.tar.gz from
http://www.belkin.com/us/support-article?articleNum=51238.

The .tar.gz file was corrupt and I couldn't find any full, good version online.  The archive still was able to extract most (?) of the files.

A buildroot Linux build system was used, and it contained the needed information on compiler and libraries used on the device.

The code is old, from 2006, so some links were broken and there were some compilation errors on my modern Ubuntu environment.  I was able to fix the links and errors and compile a toolchain successfully.

I was also able to compile alsa libraries, which may be useful in getting audio to play, and strace, which may be useful in finding out how the Skype app works with the screen and buttons.

I uploaded my changes here:

https://github.com/lawm/skype-phone

### Hello World

The device contains a hello_world in /bin:
```
# hello_world
hello_world built Aug  3 2009 17:33:14
```

But I wanted to compile my own.

With my new toolchain, I created a simple Hello World C program and compiled it.

```
PC:~/apps$ make
../patched/buildroot//buildroot-20060918/build_mips_nofpu_onephone/staging_dir/mips32/bin//mips-linux-gcc     hello.c   -o hello

```

I ran a temporary web server on my PC and used wget on the phone to retrieve the executable.

```
PC:~/apps$ python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

```
# cd /tmp
# wget http://192.168.0.123:8000/hello
# chmod 777 hello
```

... and it ran!

```
# ./hello
Hello World!
```

Now, I can check an item off my list:
- [x] be able to compile and my own code
- [ ] draw to the screen
- [ ] play audio through the speaker or handset
- [ ] read the input keys

### Draw to screen attempt

I tried to write to the framebuffer device, /dev/fb0, using cat and dd, but it results in a kernel panic.  Using an app, mmaping the framebuffer works, but nothing changes on the screen.

The Skype app, skyclient and skyhost seem to be written using Qt/Embedded for UI, and DirectFB for graphics.

And unfortunately, the GPL source package doesn't include the kernel source (or is in the corrupt part).

### Play audio attempt

I built my the alsa utility, aplay, but it couldn't find a default PCM device, or was not configured.  The device also did not come with libasound.so, so I thought maybe alsa isn't used.

    # LD_LIBRARY_PATH=.  /nvdata/aplay -l
    **** List of PLAYBACK Hardware Devices ****
    ALSA lib control.c:910:(snd_ctl_open_noupdate) Invalid CTL hw:0
    aplay: device_list:231: control open (0): No such file or directory

There is a ept.ko kernel driver, which is also used by the Skype app.  It seems to be called the Endpoint driver, used for audio.

After some Googling, I tried adding a default alsa.conf (from [stackoverflow/superuser](https://superuser.com/questions/564145/alsa-minimal-configuration)) and [alsa-project wiki](https://www.alsa-project.org/main/index.php/Asoundrc), and I was able to list the devices, and get closer to playing

    # cat alsa.conf
    ctl.hw {
       @args [ CARD ]
       @args.CARD {
           type string
       }
       type hw
       card $CARD #with 0 alsamixer work, with $CARD alsamixer lend to invalid argument
    }
    pcm.!default {
        type hw
        card 0
    }
    ctl.!default {
        type hw
        card 0
    }

    # LD_LIBRARY_PATH=. ALSA_CONFIG_PATH=/tmp/alsa.conf /nvdata/aplay -l
    **** List of PLAYBACK Hardware Devices ****
    card 0: Broadcom [Broadcom], device 0: Broadcom PCM [Broadcom PCM]
      Subdevices: 1/1
      Subdevice #0: subdevice #0

    # LD_LIBRARY_PATH=. ALSA_CONFIG_PATH=/tmp/alsa.conf /nvdata/aplay  ./audiocheck.net_sweep_250Hz_800Hz_-3dBFS_3s.wav
    Playing WAVE './a.wav' : Signed 16 bit Little Endian, Rate 44100 Hz, Mono
    aplay: set_params:882: Broken configuration for this PCM: no configurations available

Alsa configuration is pretty complicated.  I still need to mess with it more.  Maybe I need to try the whole default config instead of copy/pasting snippets from the internet.

### Read input keys attempt

After building busybox with more features, nothing was read out, but I was able to see the handset hook state and keypad events printed by the kernel:

    # /nvdata/busybox hexdump /dev/keypad0
    hook state:1
    HOOK press
    hook state:0
    HOOK release

    KEY_DIGIT_2 press
    KEY_DIGIT_2 release
    KEY_DIGIT_5 press
    KEY_DIGIT_5 release
    KEY_DIGIT_3 press
    KEY_DIGIT_3 release

### Analyzing the existing application

On startup, the phone will execute /etc/rcS.  This runs skyhost in the background, and then runs skyclient.  skyclient and skyhost communicate through a socket, /tmp/skyhost.socket.

I had problems using strace on skyhost.  with the `-f` follow forks option, skyhost would hang.  So, I had to run skyhost, wait a while, then attach to it with strace.  This way, I missed some early open() and setup calls, but I was able to see some of the normal operations.

#### Input
For input, skyclient does this:
```
open("/dev/keypad0", O_RDONLY|O_NONBLOCK) = 16
read(16, 0x7fff27c0, 1)           = -1 EAGAIN (Resource temporarily unavailable)
_newselect(18, [3 16 17], [], [], {0, 194817}) = 0 (Timeout)
gettimeofday({946686875, 11550}, NULL) = 0
...
_newselect(18, [3 16 17], [], [], {0, 193705}) = 1 (in [16], left {0, 41000})
read(16, "N", 1)                  = 1
read(16, 0x100339a1, 1)           = -1 EAGAIN (Resource temporarily unavailable)
gettimeofday({946686875, 174592}, NULL) = 0
write(17, "\0\0\0X", 4)           = 4
write(17, "#32  X-CALL VOICEENGINE VoiceCom"..., 88) = 88
read(17, "\0\0\0\22", 4)          = 4
read(17, "#32 X-REPLY 0=\"OK\"", 18) = 18
```

It opens the keypad device node, waits for data to be available with select(), then reads the button state, represented by a single character, in this case "N".  Then it sends a message to skyhost to presumably play a click sound.

#### Sound
When skyhost receives the above message, it writes a WAV file dat /dev/ept0 to play it.

- fd 4 is the skyclient to skyhost communication socket
- fd 8 is the WAV file reading fd
- fd 5 is /dev/ept0

```
read(4, "#37  X-CALL VOICEENGINE VoiceCom"..., 88) = 88
open("/skyclient/snd/click8kHz.wav", O_RDONLY) = 8
read(8, "RIFF\16\6\0\0WAVEfmt \20\0\0\0\1\0\1\0@\37\0\0\200>\0\0"..., 36) = 36
lseek(8, 0, SEEK_CUR)             = 36
read(8, "data\270\5\0\0", 8)      = 8
ioctl(5, EVIOCGVERSION, 0x7fa67370) = 0
ioctl(9, 0x89f0, 0x7fa672f8)      = -1 EOPNOTSUPP (Operation not supported)
ioctl(6, 0xc0044180, 0x2)         = 0
ioctl(6, 0xc0044170, 0x1)         = 0
read(8, "E\0\33\0;\0\35\0\311\377\325\377\363\377\352\0\273\376"..., 640) = 640
write(5, "E\0\33\0;\0\35\0\311\377\325\377\363\377\352\0\273\376"..., 640) = 640
```
I'm guessing there is initialization and configuration calls for /dev/ept0 before writing the WAV to it, but I haven't captured it.

I tried to do it anyways, but it didn't result in sound being played:
`/nvdata/busybox dd if=click8kHz.wav of=/dev/ept0`


#### Display

On skyclient start, it opens /dev/fb0, gets screen info, then mmaps it:
```
open("/dev/fb0", O_RDWR)          = 8
ioctl(8, FBIOGET_FSCREENINFO, 0x7fff22f8) = 0
ioctl(8, FBIOGET_VSCREENINFO, 0x7fff2340) = 0
brk(0x1002c000)                   = 0x1002c000
brk(0x1002d000)                   = 0x1002d000
mmap(NULL, 40960, PROT_READ|PROT_WRITE, MAP_SHARED, 8, 0) = 0x2aaae000
```

I used an app to do similar thing, but writing to the mmap'ed buffer did not result in any change in the screen.

I also saw this being done:
```
open("/dev/lcd", O_RDWR)          = 7
open("/proc/sys/fb/refresh-on", O_WRONLY) = 8
write(8, "0\n", 2)                = 2
close(8)                          = 0
```

I tried `echo 1 > /proc/sys/fb/refresh-on`, but it did not help.


### Input working!

Somehow doing `cat` on the device didn't work before, but after writing a test app to simulate the original app's behavior (and struggling with `select()`'s `nfds` argument ... which should be named maxfd), I was able to read the input button events:

    # ./input
    Data available
    r=1 buf=4e N
    r=-1 buf=4e N
    Data available
    r=1 buf=46 F
    r=-1 buf=46 F
    Data available
    r=1 buf=32 2
    r=-1 buf=32 2

and checking it off the list:
- [x] be able to compile and my own code
- [ ] draw to the screen
- [ ] play audio through the speaker or handset
- [x] read the input keys


### Display working

I could write to the framebuffer, but didn't see any of my changes on the screen.

The kernel and driver source should be in the GPL source package, but it's corrupt from Belkin, so the kernel is missing.

I searched for "/proc/sys/fb/refresh-on", one of the paths written by the app, and I found source for another device, containing the lcdfb driver ([github](https://github.com/prototype-U/Ace-i/blob/master/common/drivers/video/broadcom/fb/lcdfb.c)).

Then, I remembered the strace log had some commands sent to /dev/lcd.  I thought they were related to LCD brightness and turning the display on/off.

```
open("/dev/lcd", O_RDWR)          = 7
open("/proc/sys/fb/refresh-on", O_WRONLY) = 8
write(8, "0\n", 2)                = 2
close(8)                          = 0
...
ioctl(7, _IOC(_IOC_WRITE, 0x4c, 0x8c, 0x08), 0x1001fc28) = 0
...
ioctl(7, _IOC(_IOC_WRITE, 0x4c, 0x8c, 0x08), 0x1001fc28) = 0
```

I matched the ioctl command to

`#define LCD_IOCTL_DIRTY_ROWS                    _IOW(LCD_MAGIC, LCD_CMD_DIRTY_ROWS, LCD_DirtyRows_t)`

from other drivers for other BCM devices

https://github.com/zecn/GT-S5360-opensource-Update--3/blob/06adb992f4b6c7bcb3654a17bbbd9fd8d7308c85/common/include/linux/broadcom/lcd.h

https://github.com/rajamalw/galaxy-s5360/blob/master/kernel/drivers/video/broadcom/dss/bcm215xx/lcdc.c

The ioctl indicates which rows are dirty and need to be sent to the LCD.

I added code to call the dirty rows ioctl to my framebuffer test app:

```C
static void refresh(int lcdfd) {
    int r;
    unsigned int dirtyrows[2] = {0, 159}; // LCD_DirtyRows_t
    if (lcdfd < 0) {
        printf("lcdfd err \n");
        return;
    }
    r = ioctl(lcdfd, 0x80084c8c, &dirtyrows);
    if (r)
        printf("ioctl r=%d\n", r);
}
```

and I was able to see my changes to the framebuffer!

![skype-phone-tux](/images/2018/06/skype-phone-tux.png)

It's a bad picture, but in real life, the colors are still off (more red than orange), but it's good enough for now.

And updating the list:
- [x] be able to compile and my own code
- [x] draw to the screen
- [ ] play audio through the speaker or handset
- [x] read the input keys

### Audio attempt 2
I was able to get past some alsa configuration errors, but got a different error. I couldn't figure out what formats are supported.

```
# LD_LIBRARY_PATH=/nvdata ALSA_CONFIG_PATH=/tmp/alsa/alsa.conf:/tmp/asound.rc /tmp/aplay  -D hw:0,0  /nvdata/skyclient/snd/login8kHz.wav
Playing WAVE '/nvdata/skyclient/snd/login8kHz.wav' : Signed 16 bit Little Endian, Rate 8000 Hz, Mono
aplay: set_params:904: Sample format non available
```

I try to play a S16\_BE format to match hw params, but it has some other error:
```
# LIBASOUND_DEBUG=1 LD_LIBRARY_PATH=/nvdata ALSA_CONFIG_PATH=/tmp/alsa/alsa.conf:/tmp/asound.rc /tmp/aplay -f S16_BE /tmp/output.raw
Playing raw data '/tmp/output.raw' : Signed 16 bit Big Endian, Rate 8000 Hz, Mono
hw params default
ACCESS:  MMAP_INTERLEAVED RW_INTERLEAVED
FORMAT:  S16_BE
SUBFORMAT:  STD
SAMPLE_BITS: 16
FRAME_BITS: [16 32]
CHANNELS: [1 2]
RATE: [8000 32000]
PERIOD_TIME: [500 4096000]
PERIOD_SIZE: [16 32768]
PERIOD_BYTES: [64 65536]
PERIODS: [1 1024]
BUFFER_TIME: [500 4096000]
BUFFER_SIZE: [16 32768]
BUFFER_BYTES: [64 65536]
TICK_TIME: 1000
ALSA lib pcm.c:873:(snd_pcm_sw_params) params->avail_min is 0
aplay: set_params:1021: unable to install sw params:
start_mode: EXPLICIT
xrun_mode: NONE
tstamp_mode: NONE
period_step: 1
sleep_min: 0
avail_min: 0
xfer_align: 3
silence_threshold: 0
silence_size: 0
boundary: 1073741824
```

I look at the strace logs again and find it operates on /dev/ept0 and /dev/halaudio.  It runs some ioctl commands on /dev/halaudio before writing the WAV data to /dev/ept0.

I found a header file for the halaudio driver that matches partly, but it doesn't contain command 0x80 and 0x70 that is used on my device:

https://android.googlesource.com/kernel/bcm/+/android-bcm-tetra-3.10-lollipop-wear-release/include/linux/broadcom/halaudio_ioctl.h#147

And some other versions of the halaudio driver were found:

https://github.com/CoolDevelopment/VSMC-i9105p/blob/aefe33806b669dc4be8db00b9475291ccbba018a/drivers/char/broadcom/halaudio/halaudio_api.c

https://github.com/shuishang/kernel_bcm5892/blob/c0809c73a4ae9f5a54404b144f0c782ed05d23a8/drivers/char/broadcom/halaudio_drivers/alsa/snd_halaudio.c


### To be continued

I think I can replay the app's calls to /dev/ept0 and /dev/halaudio to play audio, but I would rather have alsa working.

At this point, the amount of work to continue to get basic things working is more than I'm willing to put in at this time, so I'm shelving the unit for now.
