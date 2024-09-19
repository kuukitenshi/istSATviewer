## UPDATE

Preliminar TLE

```
ISTSAT1P             
1 60238U 24128D   24212.97000557  .00007963  00000+0  70096-3 0  9999
2 60238  61.9950  88.5744 0010744 296.4076  63.5909 14.97152488  3168
```

# istSatViewer
Simple program that will provide the capability for any user with an SDR to receive and decode messages sent from ISTSat-1. There are two versions, one lighter for Linux users mainly and another using a VM for Windows and Mac OS X users.

For Linux it is  distributed as two docker images that will be run using docker-compose. One image is responsible for running GNURADIO and the other image is responsible for running the software that will decode the messages and print the data to the terminal. Docker was chosen in order to facilitate the development process and make it accessible to anyone. After installing Docker and Docker Compose, with just one command you should be receiving messages from the Satellite.


# Instructions for Windows and Mac OS
In Windows and Mac OS, docker does not support easily sharing the host USB device with the containers. To facilitate installation and guarantee future compatibility, we created a virtual machine (VM) that will contain everything you need to receive and decode messages from the satellite.

1. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
2. Download the ova file from [here](https://drive.google.com/drive/folders/1FTXfsTDHjU9etDDFKRuVthNt_m1gdBOq?usp=sharing)
3. Open VirtualBox
4. Click on File -> Import Appliance
5. Select the ova file you downloaded (this might take a while to complete, please be patient)
6. Please check that you are sharing the USB device with the virtual machine (this can be done by clicking on the virtual machine -> settings -> USB -> add the SDR device)
6. Start the virtual machine (better to choose Scalled Mode, View -> Host+C) 
8. login with the default username and password (username: isat password: isat)
9. Go to settings and adjust the keyboard layout to match your own

Now you should have everything you need to start receiving messages from the satellite. To start the program, simply open the terminal and run the following command:
```bash
./launcher.sh
```

This will automatically launch the gnuradio script, the decoder, and the server. The default script is configured to work with RTL-SDR. To use PlutoSDR, you need to change the script name in the config file

The config file contains the default values that will be used to launch the gnu radio script and the decoder. If you want to change any of the default values, you can edit this file to suit your needs.

# Instructions Ubuntu

1. Install docker in your system,
    - [Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)
2. Install docker compose
    - [Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)
3. Clone this repository
4. Open a terminal inside this repository
6. Connect the SDR to your computer
5. run ```docker compose up```
    - if you installed docker compose via apt, it probably installed V1, meaning that you have to run ```docker-compose up``` instead

After this you should any received messages will be shown in your terminal



## Currently Supported Radios

The current supported list of SDRs is:
- PlutoSDR
- RTL-SDR

It is also possible to use a conventional HAM Radio through:
- TNC
- Virtual TNC (audio playback/radio audio input)

the default script is configured to work with RTL-SDR. To use with PlutoSDR, you jaust have to change the docker-compose.yaml to use the correct script
the line is commented, so all you have to do is comment the rtl line and uncomment the pluto line

## TNC support

This is an alternative way to receive messages from the satellite. It avoid the need of an SDR, therefore it also avoids the need for GNU Radio.

In order to use a TNC, there are a few changes that you need to do. The file "config_tnc_example" contains and example for a typicall TNC. Make sure that you change the serial port and baud rate to the correct values.

If you are using docker and want to use it with a TNC, to minimize dependencies, you will need to run the tnc_proxy.py script locally. With the following commands:

```bash
pip install pyserial
pip install kiss
pip install kiss2
python3 scripts/tnc_proxy.py
```
Please make sure that you also comment out the GNU Radio service in the docker compose file to avoid conflicts.

The virtual machine already contains all the files and dependencies needed to run the TNC. Just make sure you update the config file correctly and if you are using a real TNC make sure that you pass the USB device to the virtual machine.

After this everything should run smoothly.

ps: when the tnc_proxy.py script starts, you should see the lights on the tnc blink three times

## Virtual TNC support
We have just added support for a virtual TNC. The software choosen to perform this task is Direwolf. This software is capable of emulating a TNC and can be used to receive messages from the satellite.

This allows a couple of different scenarios:
- playback audio recordings from the satellite to try and decode them
- use the audio output from a radio connected to your computer to try and decode the messages
- use GQRX + SDR as an alternative way to receive the messages

When the Virtual Machine boots, it will automatically create an audio sink called DireWolfSink. This sink will be the default source for the Direwolf software. To run the software, you just need to choose the correct serial port in the config file (/tmp/kisstnc). The launch script will then auotmatically start the Direwolf, pipe DirewolfSink to Direwolf and launch the decoding software.

### Audio playback
1. Add the audio file to your virtual machine. 
2. Open the audio with any audio player
3. Using a tool like pavucontrol, change the output of the audio player to DireWolfSink
4. Run the ./launch.sh script

A known  good audio file is included in the repository, you can use it to test the setup.

### GQRX + SDR
1. Open GQRX and Connect to your SDR
2. Set the correct frequency and mode (Narrow FM)
3. Using a tool like pavucontrol change the output of GQRX to DireWolfSink
4. Run the ./launch.sh script

### Custom audio source
This could be either a radio connected to your computer or a microphone for example...

To check the available audio sources you can use the following command:
```
pactl list sources short 
```

After you have identified the correct source, you can change the default source. For this you will need to open scripts/tnc_wrapper.py and change the source name (the default name is DireWolfSink.monitor) there are some comments in the script to help guide you

With the selected flags, every 5 seconds Direwolf will print to terminal some informaiton about the audio received (audio level and sample rate). This can be useful to check that Direwolf is actually receiveing the audio. However by default this it piped do /dev/null, so if you are having problems and want to check that everything is okay, you can remove the redirection to /dev/null in the launch.sh script. You might also want to comment out the launch of the decoder and viewing page to make sure nothing is cluttering the terminal.

## Operational parameters

**Downlink Frequency**: 145.895 MHz \
**CallSign**: CT6IST 

# Common Errors

```console
gnuradio_nui_1  | Traceback (most recent call last):
gnuradio_nui_1  |   File "istsat_radio_v2/AFSK.py", line 971, in <module>
gnuradio_nui_1  |     main()
gnuradio_nui_1  |   File "istsat_radio_v2/AFSK.py", line 959, in main
gnuradio_nui_1  |     tb = top_block_cls(rx_offset=options.rx_offset, tx_offset=options.tx_offset, uri=options.uri)
gnuradio_nui_1  |   File "istsat_radio_v2/AFSK.py", line 716, in __init__
gnuradio_nui_1  |     self.pluto_source_0 = iio.pluto_source(uri, 145895000-sample_rate/4+rx_offset, 32*sample_rate, 32*sample_rate, 0x8000, True, True, True, "slow_attack", 55, '', True)
gnuradio_nui_1  |   File "/usr/lib/python2.7/dist-packages/gnuradio/iio/iio_pluto_source_swig.py", line 126, in make
gnuradio_nui_1  |     return _iio_pluto_source_swig.pluto_source_make(*args, **kwargs)
gnuradio_nui_1  | RuntimeError: Unable to create context
```

If you see something similar to this means that the container is not able to access the SDR. There could be many issues that could be causing this. Please check that you have the SDR correctly connected and configured for your computer, make sure that you are using the correct ip or correct port.
