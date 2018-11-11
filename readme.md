# Pi-BNO055

## Background

This is a C driver program for operating a Bosch BNO055 IMU sensor via I2C on a Raspberry Pi. I used it with a GY-BNO055 and a Adafruit BNO055. On the GY-BNO055, I had to bridge two solder pads for enabling I2C mode, because serial mode was default.  Later I switched to Adafruit for the superior quality and the onboard 5V-level support.

<img src="ada-bno055.png" height="320px" width="273px">

## I2C bus connection

Connecting the GY-BNO055 sensor to the Raspberry Pi I2C bus, the sensor responds with the slave address 0x29. The Adafruit sensor responds by default under 0x28.

```
root@pi-ws01:/home/pi# i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- 28 -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

## Code compilation

Compiling the test program:
````
root@pi-ws01:/home/pi/bno055# make
cc -O3 -Wall -g   -c -o i2c_bno055.o i2c_bno055.c
cc -O3 -Wall -g   -c -o getbno055.o getbno055.c
cc i2c_bno055.o getbno055.o -o getbno055
````

## Example output

Running the program, extracting the sensor version and configuration information:
```
pi@nanopi-neo2:~/pi-bno055 $ ./getbno055 -t inf

BN0055 Information at Sun Nov 11 14:09:47 2018
----------------------------------------------
   Chip Version ID = 0xA0
  Accelerometer ID = 0xFB
      Gyroscope ID = 0x0F
   Magnetoscope ID = 0x32
  Software Version = 3.17
   Operations Mode = NDOF_FMC_OFF
        Power Mode = NORMAL
Axis Configuration = X==X Y==Y Z==Z (ENU)
   Axis Remap Sign = X+ Y+ Z+
System Status Code = Sensor running with fusion algorithm
Accelerometer Test = OK
 Magnetometer Test = OK
    Gyroscope Test = OK
MCU Cortex M0 Test = OK
 System Error Code = No Error
Acceleration Unit  = m/s2
    Gyroscope Unit = dps
        Euler Unit = Degrees
  Temperature Unit = Celsius
  Orientation Mode = Windows
Sensor Temperature = 29°C

----------------------------------------------
Accelerometer  Power = NORMAL
Accelerometer Bwidth = 7.81Hz
Accelerometer GRange = 2G
Accelerometer  Sleep = event-driven, 0.5ms

----------------------------------------------
Sensor System Calibration = Fully calibrated
    Gyroscope Calibration = Fully calibrated
Accelerometer Calibration = Minimal Calibrated
 Magnetometer Calibration = Fully calibrated
```

Running the program, showing the sensor calibration state and offset values:
```
pi@nanopi-neo2:~/pi-bno055 $ ./getbno055 -t cal
sys [S:3] acc [S:1 X:0 Y:65534 Z:65528 R:1000] mag [S:3 X:65428 Y:65424 Z:65476 R:656] gyr [S:3 X:65534 Y:0 Z:1]
```

Changing the operational mode, e.g. to CONFIG:
```
pi@pi-ws01:~/pi-bno055 $ ./getbno055 -v -m config
Debug: arg -s, value config
Debug: ts=[1539005771] date=Mon Oct  8 22:36:11 2018
Debug: Sensor Address: [0x28]
Debug: Write opr_mode: [0x00] to register [0x3D]
```

Resetting the sensor:
```
pi@pi-ws01:~/pi-bno055 $ ./getbno055 -v -r
Debug: arg -r, value (null)
Debug: ts=[1539005864] date=Mon Oct  8 22:37:44 2018
Debug: Sensor Address: [0x28]
Debug: BNO055 Sensor Reset complete
```

NDOF fusion mode, Euler angles
```
pi@nanopi-neo2:~/pi-bno055 $ ./getbno055 -t eul
EUL 233.00 -3.12 -15.94
```

Writing calibration data to file
```
pi@nanopi-neo2:~/pi-bno055 $ ./getbno055 -t cal -w bno.cfg
sys [S:3] acc [S:1 X:0 Y:65534 Z:65528 R:1000] mag [S:3 X:65428 Y:65424 Z:65476 R:656] gyr [S:3 X:65534 Y:65535 Z:1]

pi@nanopi-neo2:~/pi-bno055 $ ls -l bno.cfg
-rw-rw-r-- 1 pi pi 22 Nov 11 14:17 bno.cfg

pi@nanopi-neo2:~/pi-bno055 $ od -A x -t x1 -v bno.cfg
000000 00 00 fe ff f8 ff 94 ff 90 ff c4 ff fe ff ff ff
000010 01 00 e8 03 90 02
000016
```
## Usage

Program usage:
```
pi@nanopi-neo2:~/pi-bno055 $ ./getbno055
Usage: getbno055 [-a hex i2c-addr] [-m <opr_mode>] [-t acc|gyr|mag|eul|qua|lin|gra|inf|cal] [-r] [-w calfile] [-l calfile] [-o htmlfile] [-v]

Command line parameters have the following format:
   -a   sensor I2C bus address in hex, Example: -a 0x28 (default)
   -m   set sensor operational mode. mode arguments:
           config   = configuration mode
           acconly  = accelerometer only
           magonly  = magnetometer only
           gyronly  = gyroscope only
           accmag   = accelerometer + magnetometer
           accgyro  = accelerometer + gyroscope
           maggyro  = magetometer + gyroscope
           amg      = accelerometer + magnetometer + gyroscope
           imu      = accelerometer + gyroscope fusion -> rel. orientation
           compass  = accelerometer + magnetometer fusion -> abs. orientation
           m4g      = accelerometer + magnetometer fusion -> rel. orientation
           ndof     = accelerometer + mag + gyro fusion -> abs. orientation
           ndof_fmc = ndof, using fast magnetometer calibration (FMC)
   -p   set sensor power mode. mode arguments:
          normal    = required sensors and MCU always on (default)
          low       = enter sleep mode during motion inactivity
          suspend   = sensor paused, all parts put to sleep
   -r   reset sensor
   -t   read and output sensor data. data type arguments:
           acc = Accelerometer (X-Y-Z axis values)
           gyr = Gyroscope (X-Y-Z axis values)
           mag = Magnetometer (X-Y-Z axis values)
           eul = Orientation E (H-R-P values as Euler angles)
           qua = Orientation Q (W-X-Y-Z values as Quaternation)
           gra = GravityVector (X-Y-Z axis values)
           lin = Linear Accel (X-Y-Z axis values)
           inf = Sensor info (23 version and state values)
           cal = Calibration data (mag, gyro and accel calibration values)
*  -l   load sensor calibration data from file, Example -l ./bno055.cal
   -w   write sensor calibration data to file, Example -w ./bno055.cal
   -o   output sensor data to HTML table file, requires -t, Example: -o ./bno055.html
   -h   display this message
   -v   enable debug output

Note: The sensor is executing calibration in the background, but only in fusion mode.

Usage examples:
./getbno055 -a 0x28 -t inf -v
./getbno055 -t cal -v
./getbno055 -t eul -o ./bno055.html
./getbno055 -m ndof
./getbno055 -w ./bno055.cal

```
