# uBlox NEO-M8U Handler for use with Chasemapper
Hey look! Yet another partial implementation of the uBlox binary protocol. 

The uBlox NEO-M8U is a new (2021) GPS from uBlox which incorporates an accelerometer and gyroscope for what uBlox calls 'Untethered Dead Reckoning'. The short version is that we end up with a GPS that can fuse multi-GNSS data and the IMU sensor data, for much better positional accuracy. More importantly for [Chasemapper](https://github.com/projecthorus/chasemapper) & Amateur Radio Direction-Finding (ARDF) use, the NEO-M8U can provide an accurate car heading, even when the car is moving slowly. This is extremely useful for fusion and display of incoming bearing data.

This repository is my attempt at a 'driver' for the uBlox NEO-M8U, to get the position and heading data into a format suitable for use with Chasemapper. It'll talk to a serial-connected (USB or otherwise) uBlox NEO-M8U, configure it, and emit UDP messages in a format Chasemapper can ingest.

I may eventually merge this into Chasemapper itself.

### Contacts
* [Mark Jessop](https://github.com/darksidelemm) - vk5qi@rfhead.net

## Basic Operation
### Dependencies
* Python >=3.6
* pyserial package (from pip)

### Startup
Usage:
```
$ python ubloxm8u.py --help
usage: ubloxm8u.py [-h] [-v] [--ntp] [--udp_port UDP_PORT] port

positional arguments:
  port                 uBlox NEO-M8U Serial Port

optional arguments:
  -h, --help           show this help message and exit
  -v, --verbose        Verbose output.
  --ntp                Update a local NTPD Server with time data.  (Not yet tested)
  --udp_port UDP_PORT  UDP Port to Emit UDP messages on.
```

Example:
```
$ python ubloxm8u.py /dev/tty.usbmodem1421
```

### Running on Startup
One option is to start up the script in a screen session on boot, by adding the following to /etc/rc.local:
```
su - pi -c "screen -dm -S ubloxm8u python3 /home/pi/chasemapper-gps-m8u/ubloxm8u.py -v /dev/tty.usbmodem1241"
```

TODO: Systemd service.

## IMU Alignment
The uBlox NEO-M8U has an internal IMU (Accelerometer & Gyro), and will automatically perform an alignment when the GPS is started. You need to perform a few sharp left-right turns for it to figure itself out. Alignment status is emitted in the UDP message and displayed on Chasemapper's GUI (if headiing display is enabled).

### Chasemapper Output Format
The following JSON object is emitted by UDP broadcast on port 55672 by default (configurable on the command-line)
```
{ 
    'type': 'GPS',
    'latitude' : -34.123456,
    'longitude' : 138.123456,
    'altitude': 100.123456,
    'speed': 0.00000,               # Speed in kph
    'heading_status': "Unknown",    # This starts out as "Unknown", then ends up at "Coarse/Fine IMU-Mount Alignment"
    'heading': 51.05,               # This is only included if fused heading data is available
}
```


