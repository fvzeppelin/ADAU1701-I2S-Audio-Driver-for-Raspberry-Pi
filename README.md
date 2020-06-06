# ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi
Generic audio driver to use the I2S interface of the Raspberry Pi for sound output to a dsp

This is a fork of this repo:
https://github.com/MKSounds/ADAU1701-I2S-Audio-Driver-for-Raspberry-Pi

I exchanged the spdif-transmitter driver with the pcm5102a driver to make it work with piCorePlayer.

## Wiring and ADAU1701 configuration:
From
https://digital-audio-labs.jimdofree.com/english/raspberry-pi/adau1701-i2s-driver/

In brief:
###  Wiring Raspberry to ADAU1701
* Pin 12 (PCM_CLK)  -  MP5 and MP11
* Pin 35 (PCM_FS)  -  MP4 and MP10
* Pin 40 (PCM_DOUT)  -  MP0 (or MP1, MP2, MP3)
* Pin 39 (GND)  -  any GND-Pin 

### Configuration - ADAU1701
* Sample rate: 48 kHz
* Bit depth: 24 bit
* Data format: I2S
<!-- -->
* Clock master at the I²S output!
* MCLK = 256 * Fs = 12.288 MHz
* BCLK = 64 * Fs = 3.072 MHz  --> Internal / 16
* LRCLK = Fs = 48 kHz  --> Internal / 1024
<!-- -->
* LRCLK polarity: falling edge (LRCLK = 0 = left channel, LRCLK = 1 = right channel)
* BCLK data change: falling edge
* Transmission: MSB first
* MSB position: delayed by 1 BCLK 

## Installation in piCorePlayer
(as documented here:
https://forums.slimdevices.com/showthread.php?110964-ADAU1701-with-piCorePlayer&p=977571&viewfull=1#post977571)

### Device Tree Overlay
* Copy `adau1701-i2s.dtbo` to `boot/overlays/`
* Change the content of `boot/config.txt`:
  * Search the following line and comment with hash:
`#dtparam=audio=on`
  *  Add the following line at the end of the file:
`dtoverlay=adau1701-i2s`

After reboot, `aplay -l` should output something like:

```
**** List of PLAYBACK Hardware Devices ****`
card 0: Output [ADAU1701 I2S Output], device 0: bcm2835-i2s-pcm5102a-hifi pcm5102a-hifi-0 [bcm2835-i2s-pcm5102a-hifi pcm5102a-hifi-0]
Subdevices: 0/1
Subdevice #0: subdevice #0
```
### Squeezelite configuration:
* Audio output device settings: Analog audio
* Output setting (-o) -> `sysdefault:CARD=Output`
* ALSA setting (`<b>:<p>:<f>:<m>:<d>`) -> `80::24:1:`
* Max sample rate (-r) -> 48000
* Upsample setting (-R) -> vLX

### Optional: adapt ALSA config file
Add the following lines to the ALSA config file `/etc/asound.conf`:
```
pcm.InterpolatedOutput {
        type plug
        slave {
                pcm "hw:0,0"
                format S24_LE
                rate 48000
        }
}
pcm.!default InterpolatedOutput 
```
