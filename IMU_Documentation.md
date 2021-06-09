# Inertial Measurement Unit (IMU) Sensor Operation
## Background
The inertial measurement unit used was the SparkFun ICM-20948, which includes an 3-axis accelerometer, gyroscope, and magentometer. This device was used over the GY-521, which only includes a 3-axis accelerometer and gyroscope. 

The reasons for this was to properly calculate the yaw angle for the device, using sensor fusion. Normally calculating the yaw angle requires the integration of the yaw rate from the gyroscope. However, the integration causes the error in the sensor to exponentially increase causing overall drift in the calculated yaw angle. 

The ICM20948_Sensor_Usage.ino script takes advantage of the technique introduced in the *Data Fusion with 9 Degrees of Freedome Intertial Measurement Unit To Determine Object's Orientation* [1] paper writen by Long Tran at the California Polytechnic State University to calculate yaw, pitch, and roll accurately. <br/>

## Included Libraries
The ICM20948_Sensor_Usage.ino script uses the TKJ Electronics Lauszus Kalman Filter library for data filtering. This library can be downloaded through the Arduino IDE library manager or downloaded from their GitHub page [2].

Finally, using the SparkFun ICM-20948 IMU requires the SparkFun library for the ICM-20948 found in the Arduino IDE library manager or at the SparkFun GitHub page [3].<br/>

## Device Setup
Communication with the IMU was done via I2C communication protocol. The pinouts for the ICM-20948 device can be seen in the graphic below for the I2C side:

<p align="center">
  <img width="350" height="350" src=images/ICM_20948_I2C.png>
</p>

For I2C communication the Vcc pin was connected to the 5V supply, the GND pin was connected to GND, the SCL pin was connected to pin A21, and finally the SDA pin was connected to pin A20. Pins A21 and A20 are the default SCL and SDA pins on the Arduino MEGA 2560, while pins A5 and A4 are the default SCL and SDA pins on the Arduino Uno. Below is a picture showing the wire connections on the actual board: 

<p align="center">
  <img width="450" height="450" src=images/ICM_20948_Physical_Connect.jpg>
</p>

## Device Testing Based on Reference Excel Files
To run similar tests to the included Excel data files the IMU must first be on a level surface and at rest (not moving). <br/>

### Digital Low Pass Filter (DLPF) Details
In order to toggle wheter the digital low pass filters for the accelerometer and gyroscope are used the enaccDLPF and engyrDLPF booleans must be set within the ICM20948_Sensor_Usage.ino script. Both DLPF booleans were set to true for testing, but the results using the IMU built-in DLPF were very similar to the results that didn't use the built-in DLPF function. 

For details about the various DLPF configurations available look for the Digital Low-Pass Filter configuration section in the ICM20948_Sensor_Usage.ino script. 

For testing with DLPF enable the accelerometer DLPF was set to have a 3db bandwidth of 473 Hz and a Nyquist bandwidth of 499 Hz. The gyroscope DLPF was set to a 3db bandwidth of 361.4 Hz and a Nyquist bandwidth of 376.5 Hz for testing. <br/><br/>

### Filtering Configuration Details
The Kalman Filter can be enabled by setting the #if boolean to 0 or 1 for the "Kalman Filter Global Variables" sections at the beginning of the ICM20948_Sensor_Usage.ino script. Otherwise the data will be unfiltered. Also note that the Kalman filter only provides filtered results for yaw, pitch, and roll. The Kalman filtering **has not** been designed to filter the raw accelerations or angular velocities.<br/>

### Calibration Cycle Details
Finally the ICM20948_Sensor_Usage.ino script includes a calibration sequence at the beginning, that can be enabled and disabled within void setup() by changing the calibration's #if value from 0 to 1. The calibration cycle takes a second or so to run, so during this time do not move the IMU in any way. After the calibration is completed a message will be displayed in the Serial Monitor saying so and showing the offsets calculated, in measurement units (MU), during the calibration cycle. These offsets are meant to correct the built in error with the IMU since at rest the values of the accelerometer and gyroscope may not be exactly zero. This also helps correct for slight variations in the surface the IMU is on, since it defines this starting orientation as its universial 0 reference. <br/>

### Serial Monitor Outputs
For each test explained the output data can vary depending on what filter is used and whether the TimeData is enable or not. Additionally, not all of the data was uncommented for display, which can be changed depending on the user's need. If filters are enabled then the Serial Monitor will output the filtered data. <br/>
When the Kalman Filter is enabled the data will be organized as follows: <br/>

| Time ($\mu$sec) | Roll (deg) | Pitch (deg) | Yaw (deg) | AcX (MU) | AcY (MU) | AcZ (MU) | GyX (MU) | GyY (MU) | GyZ (MU) | MgX (MU) | MgY (MU) | MgZ (MU) | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |  ----- | ----- | ----- |

**NOTE: ALL MU DATA IS RAW DATA FROM THE IMU**<br/>
*MU stands for Measurement Units. Conversions for each value are as follows:* 
*1 g = 9.81 $\frac{m}{s^2}$ = 16384 MU for acceleration data*
*1 $\frac{deg}{s}$ = 131 MU for gyroscope data*
*1 $\mu$Tesla = 6.66 MU for magnetometer data*
*Time ($\mu$sec) is the change in time (dt) between readings* <br/>

When the Kalman filter is disabled and no filtering is done the data will be organized as follows: <br/>
| Time ($\mu$sec) | AcX (MU) | AcY (MU) | AcZ (MU) | GyX (MU) | GyY (MU) | GyZ (MU) | MgX (MU) | MgY (MU) | MgZ (MU) | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
**NOTE: ALL MU DATA IS RAW DATA FROM THE IMU**<br/>

The formating will for each of these tables will stay the same even if the calibration cycle is disabled. 

### Testing Configurations
<ins>Test 1: Yes KF and Yes Calibration
- Kalman Filtering (KF) **enabled**
`[Line 58]` `//Kalman Filter Global Variables`
`[Line 59]` `#if 1`
- Digital Low Pass Filter (DLPF) for accelerometer and gyroscope **disabled** 
`[Line 37]` `bool enaccDLPF = false;`
`[Line 38]` `bool engyrDLPF = false;`
- Calibration cycle **enabled**
`[Line 210]` `//Set to 1 in order to Enable calibration`
`[Line 211]` `#if 1`
- 5 seconds of data collection while IMU is standing still 

<ins>Test 2: : No KF and Yes Calibration
- Kalman Filtering (KF) **disabled**
`[Line 58]` `//Kalman Filter Global Variables`
`[Line 59]` `#if 0`
- Digital Low Pass Filter (DLPF) for accelerometer and gyroscope **disabled** 
`[Line 37]` `bool enaccDLPF = false;`
`[Line 38]` `bool engyrDLPF = false;`
- Calibration cycle **enabled**
`[Line 210]` `//Set to 1 in order to Enable calibration`
`[Line 211]` `#if 1`
- 5 seconds of data collection while IMU is standing still
- Roll, Pitch, and Yaw were all manually calculated by doing a running total of using the change in time and the recorded gyroscope values. (GyX for roll, GyY for pitch, and GyZ for yaw) 

<ins>Test 3: Yes KF and No Calibration
- Kalman Filtering (KF) **enabled**
`[Line 58]` `//Kalman Filter Global Variables`
`[Line 59]` `#if 1`
- Digital Low Pass Filter (DLPF) for accelerometer and gyroscope **disabled** 
`[Line 37]` `bool enaccDLPF = false;`
`[Line 38]` `bool engyrDLPF = false;`
- Calibration cycle **disabled**
`[Line 210]` `//Set to 1 in order to Enable calibration`
`[Line 211]` `#if 0`
- 5 seconds of data collection while IMU is standing still 

<ins>Test 4: No KF and No Calibration
- Kalman Filtering (KF) **disabled**
`[Line 58]` `//Kalman Filter Global Variables`
`[Line 59]` `#if 0`
- Digital Low Pass Filter (DLPF) for accelerometer and gyroscope **disabled** 
`[Line 37]` `bool enaccDLPF = false;`
`[Line 38]` `bool engyrDLPF = false;`
- Calibration cycle **disabled**
`[Line 210]` `//Set to 1 in order to Enable calibration`
`[Line 211]` `#if 0`
- 5 seconds of data collection while IMU is standing still 

<ins>Test 5: Yes KF and Yes Calibration
- Kalman Filtering (KF) **disabled**
`[Line 58]` `//Kalman Filter Global Variables`
`[Line 59]` `#if 1`
- Digital Low Pass Filter (DLPF) for accelerometer and gyroscope **disabled** 
`[Line 37]` `bool enaccDLPF = false;`
`[Line 38]` `bool engyrDLPF = false;`
- Calibration cycle **disabled**
`[Line 210]` `//Set to 1 in order to Enable calibration`
`[Line 211]` `#if 1`
- Have the IMU standing still for 2 seconds. Rotate IMU CCW about 90 degrees and hold for 2 seconds. Rotate the IMU CW 180 degrees and hold for 2 seconds. Finally, rotate the IMU CCW 90 degrees and hold for 2 seconds at around its original position. (Try your best to rotate the at a constant velocity)

### Results Comparison
Open the IMU_Testing_Reference_Data Excel.xlsx Excel spreadsheet to view the reference testing data for the IMU. 

In order to convert the raw MU data to metric units a few different conversions may be required. These conversions are explained previously in the Serial Monitor Outputs section. 

Compare the collected data to the reference data and make sure similar behavior is documented. Some things to note with this comparison are as follows: 
1. Yaw angle may not be the same when comparing since yaw is determined by global orientation. This is because the yaw calculation for the Kalman Filter is dependent on the IMU's magnetometer. 
2. Ensure that the MU to metric conversion is done properly. There may be an order of magnitude error with the conversion, which could result in data favoring the 0 axis. To check for this issue make sure to move the IMU back and forth relatively quickly for a few seconds and check the data. The reported values should be around 2-3 $\frac{m}{s^2}$. **If not** then try changing the conversion factor from 16,384 MU = 9.81 $\frac{m}{s^2}$ to 16.384 MU = 9.81 $\frac{m}{s^2}$ and check again. 
(This issue comes from pulling data directly from the device registers. For the main car control program the SparkFun ICM-20948 library functions are used, which provide results in already converted metric units)

## References
[1] [*Data Fusion with 9 Degrees of Freedome Intertial Measurement Unit To Determine Object's Orientation*](https://digitalcommons.calpoly.edu/cgi/viewcontent.cgi?article=1422&context=eesp) 

[2] [TKJ Electronics Lauszus Kalman Filter GitHub](https://github.com/TKJElectronics/KalmanFilter)

[3] [SparkFun ICM-20948 Arduino Library GitHub](https://github.com/sparkfun/SparkFun_ICM-20948_ArduinoLibrary)

[4] [SparkFun IMU Breakout Hookup Guide](https://learn.sparkfun.com/tutorials/sparkfun-9dof-imu-icm-20948-breakout-hookup-guide)

[5] [ICM-20948 9-DOF IMU Datasheet](https://invensense.tdk.com/wp-content/uploads/2016/06/DS-000189-ICM-20948-v1.3.pdf)