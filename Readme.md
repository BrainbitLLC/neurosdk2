# Documentation

## Overview

Neurosdk is a powerful tool for working with neuro-sensors BrainBit, BrainBit2, BrainBitBlack, DragonEEG, Callibri and Kolibri, Headband and Headphones. All these devices work with BLE 4.0+ technology. SDK allows you to connect, read the parameters of devices, as well as receive signals of various types from the selected device. 

## Getting Started

Firstly, you need to install package. 

Neurosdk for android is distributed using JitPack as an aar library. Here is an example of adding SDK to an AndroidStudio project using gradle:

Add to `settings.gradle` of project:

###### Groovy:
```groovy
dependencyResolutionManagement {
	repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}
```

###### Kotlin:
```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        ...
        maven { setUrl("https://jitpack.io") }
    }
}
```

and to `build.gradle` of app:

###### Groovy:
```groovy
dependencies {
    implementation 'com.github.BrainbitLLC:neurosdk2:1.0.6.18'
}
```

###### Kotlin:
```kotlin
dependencies {
    implementation("com.github.BrainbitLLC:neurosdk2:1.0.6.18")
}
```

SDK uses a runtime permission to bluetooth and location, so you need to ask user about it. For this action SDK contains helper classes:

###### Java:
```java
import com.neurosdk2.helpers.PermissionHelper;
import com.neurosdk2.helpers.interfaces.IPermissionListener;
...
if(!PermissionHelper.HasAllPermissions(ctx)){
    PermissionHelper.RequestPermissions(ctx, (grantedPermissions, deniedPermissions, deniedPermanentlyPermissions) -> {
        ...                  
    }); 
}
```
###### Kotlin:
```kotlin
if(!PermissionHelper.HasAllPermissions(requireContext())){
                PermissionHelper.RequestPermissions(requireContext()
                ) { grantedPermissions, deniedPermissions, deniedPermanentlyPermissions -> ... }
            }
```

## Usage

SDK can be conditionally divided into two parts: device search (Scanner object) and the device itself (Sensor object).

The scanner allows you to find devices nearby, it is also responsible for the first creation of the device. When created, the device is automatically connected. In the future, the connection state can be controlled through the sensor object. Whatever type of device you work with, the use of the scanner will be the same.

The sensor allows you to monitor the status of the device, set parameters, receive a signal of various types. 

>It is recommended to work with both parts in a separate thread. During the connection, the SDK also reads the main characteristics of the device, which significantly increases the time of the connection method, which will cause the application to freeze.

If an error occurs while working with the library, you will receive an exception for any of the methods.

### Errors

Here is a list of exceptions that occur when working with SDK. You need to be guided by this list in order to understand what happened in the process of executing a particular method.

| Code| Description                                             |
| ----| --------------------------------------------------------|
| 100 | Invalid scan parameters are specified                   |
| 101 | Invalid sensor types are specified for the search       |
| 102 |Failed to create sensor scanner                          |
| 103 |Failed to started sensor scanner                         |
| 104 |Failed to stopped sensor scanner                         |
| 105 |Failed to get a list of sensors                          |
| 106 |Failed to get a list of sensors                          |
| 107 |Invalid parameters for creating a sensor                 |
| 108 |Failed to create sensor                                  |
| 109 |Sensor not founded                                       |
| 110 |Failed to connect the sensor                             |
| 111 |Failed to disconnect the sensor                          |
| 112 |Failed to get a list of sensor features                  |
| 113 |Invalid parameters for get a list features of the sensor |
| 114 |Invalid parameters for get a list commands of the sensor |
| 115 |Failed to get a list of sensor commands                  |
| 116 |Invalid parameters for get a list parameters of the sensor|
| 117 |Failed to get a list of sensor parameters                |
| 118 |Failed to execute the sensor command                     |
| 119 |Failed read the sensor parameter                         |
| 120 |Failed read the sensor parameter                         |
| 121 |Failed write the sensor parameter                        |
| 122 |Failed write the sensor parameter                        |
| 123 |Failed add callback the sensor                           |
| 124 |Failed add callback the sensor                           |

### Scanner

The scanner works like this:

1. Create scanner. 
When creating a scanner, you need to specify the type of device to search. It can be either one device or several. Here is example for two type of devices - BrainBit and Callibri.

###### Java

```java
Scanner scanner = new Scanner(SensorFamily.SensorLECallibri, SensorFamily.SensorLEBrainBit);
```

###### Kotlin

```kotlin
var scanner = Scanner(SensorFamily.SensorLECallibri, SensorFamily.SensorLEBrainBit)
```

2. During the search, you can get a list of found devices using a callback. To do this, you need to subscribe to receive the event, and unsubscribe after the search is completed:

###### Java

```java
scanner.sensorsChanged = (scanner, sensors) -> {
    sensors.forEach(sensorInfo -> Log.i("TAG", sensorInfo.getName() + ": " +  sensorInfo.getAddress()));
};
...
scanner.sensorsChanged = null;
```

###### Kotlin

```kotlin
scanner?.sensorsChanged = Scanner.ScannerCallback { scanner, sensors ->
    sensors.forEach { sensorInfo -> Log.i("TAG", sensorInfo.name+ ": " + sensorInfo.address) }
}
...
scanner.sensorsChanged = null
```

3. Start search

###### Java

```java
scanner.start();
```

###### Kotlin

```kotlin
scanner.start();
```

3. Stop search

###### Java

```java
scanner.stop();
```

###### Kotlin

```kotlin
scanner.stop();
```

4. Additionally, a list of found devices can be obtained using a separate method.

###### Java

```java
List<SensorInfo> sensors = scanner.getSensors();
```

###### Kotlin

```kotlin
var sensors: List<SensorInfo>?  = scanner.sensors
```

`SensorInfo` contains information about device:
 - Name - the name of device
 - Address - MAC address of device (UUID for iOS/MacOS)
 - Serial number - device's serial number
 - Sensor family - type of device
 - Sensor model - numerical value of the device model
 - Pairing requared - whether the device needs to be paired or not

5. After you finish working with the scanner, you need to clean up the resources used. 

###### Java

```java
scanner.close();
```

###### Kotlin

```kotlin
scanner.close()
```

> Important!
> When restarting the search, the callback will only be called when a new device is found. If you need to get all devices found by the current scanner instance, call the appropriate method.


### Sensor

#### Creating

You need to create a device using a scanner. All manipulations with the device will be performed without errors only if the device is connected.

###### Java

```java
// BrainBit
BrainBit sensor = (BrainBit)scanner.createSensor(sensorInfo);

// Callibri
Callibri sensor = (Callibri)scanner.createSensor(sensorInfo);
```

###### Kotlin

```kotlin
// BrainBit
var sensor: Sensor? = scanner.createSensor(deviceInfo) as BrainBit

// Callibri
var sensor: Sensor? = scanner.createSensor(deviceInfo) as Callibri
```

> Device creation is a blocking method, so it must be called from separate thread.

> For all types of devices, you can use the same methods to control the device's connection status, invoke commands, and check for functionality.

#### Manage connection state

Connection status can be obtained in two ways. The first one is using the sensor property `State`.

The second way is in real time using a callback:

###### Java

```java
sensor.sensorStateChanged = state -> {
    Log.i("TAG", state.toString());
};
...
sensor.sensorStateChanged = null;
```

###### Kotlin

```kotlin
sensor.sensorStateChanged = Sensor.SensorStateChanged { state ->
    Log.i("TAG", state.toString())
}
...
sensor.sensorStateChanged = null
```

A connection can be in two states: connected (InRange) and disconnected (OutOfRange).

> Important!
> The state change callback will not come after the device is created, only after disconnecting (device lost from the scope or using a method) and then connected. The device does not automatically reconnect.

You can connect and disconnect from device manually by methods `Connect()` and `Disconnect()`. To receive connection state in real time you need to subscribe to `stateChanged` event. Also you can get connection state by sensor's property.

###### Java

```java
sensor.disconnect();
...
sensor.connect();
```

###### Kotlin

```kotlin
sensor.disconnect()
...
sensor.connect()

```

> Method `Connect` is blocking too, so it need to be call from separate thread.

#### Manage device parameters

##### Battery

Also, you can get power value from each device by sensor property `BattPower` or by callback in real time:

###### Java

```java
sensor.batteryChanged = battery -> {
    Log.i("TAG", "Power: " + battery);
};
...
sensor.batteryChanged = null
```

###### Kotlin

```kotlin
sensor.batteryChanged = Sensor.BatteryChanged { power ->
    Log.i("TAG", power.toString())
}
...
sensor.batteryChanged = null
```

##### Parameters

Each device has its own settings, and some of them can be configured as you need. Not all settings can be changed and not every device supports all settings.

First you need to find out what parameters the device supports and whether they can be changed:

###### Java

```java
List<ParameterInfo> parameters = sensor.getSupportedParameters();
parameters.forEach(parameter -> Log.i("TAG", parameter.getParam() + ": " + parameter.getParamAccess()));

// Output:
// 
// ParameterGain: ReadWrite  
// ParameterState: ReadNotify
```

###### Kotlin

```kotlin
val parameters = sensor.supportedParameters
parameters?.forEach { parameter ->
    Log.i(
        "TAG",
        "${parameter.param}: ${parameter.paramAccess}"
    )
}

// Output:
// 
// ParameterGain: ReadWrite  
// ParameterState: ReadNotify
```

Info about parameter includes two fields:
 - the name of the parameter, represented by an enumeration
 - parameter availability for manipulation. Can be one of three values:
   - read - read-only
   - read and write - parameter can be changed
   - read and notify - parameter is updated over time

You can also check if the parameter is supported, for example `Gain`:

###### Java

```java
if(sensor.isSupportedParameter(SensorParameter.ParameterGain)){
...
}
```

###### Kotlin

```kotlin
if(sensor.isSupportedParameter(SensorParameter.ParameterGain) == true){
...
}
```

##### Parameter description

###### Name

Name of device. String value.

###### Java

```java
String name = sensor.getName();
...
sensor.setName("NewName"); // <- this is throw an exeption, you cannot set device name
```

###### Kotlin

```kotlin
var name: String = sensor.name
...
sensor.name = "NewName" // <- this is throw an exeption, you cannot set device name
```

###### State

Information about the connection status of a device. Может принимать два значения:

- InRange - Device connected
- OutOfRange - The device is turned off or out of range

###### Java

```java
SensorState state = sensor.getState(); 
if(state == SensorState.StateInRange){
    // connected
}
else {
    // disconnected
}
```

###### Kotlin

```kotlin
var state: SensorState = sensor.state
if(state == SensorState.StateInRange){
    // connected
}
else {
    // disconnected
}
```


###### Address

MAC-address of device. For iOS/MacOS represents by UUID. String value.

###### Java

```java
String address = sensor.getAddress(); 
```

###### Kotlin

```kotlin
var address: String = sensor.address
```

###### SerialNumber

Serial number of device.  String value.

For callibri device families, this field is empty in `SensorInfo` when searching, and you can get it immediately after connecting using this property.

###### Java

```java
String serial = sensor.getSerialNumber(); 
```

###### Kotlin

```kotlin
var serial: String = sensor.serialNumber
```

###### HardwareFilterState

Device signal filter activity states. If the parameter is supported by the device, it becomes possible to set the desired filters using the `HardwareFilters` property. The next filters are available:

- HPFBwhLvl1CutoffFreq1Hz
- HPFBwhLvl1CutoffFreq5Hz
- BSFBwhLvl2CutoffFreq45_55Hz
- BSFBwhLvl2CutoffFreq55_65Hz
- HPFBwhLvl2CutoffFreq10Hz
- LPFBwhLvl2CutoffFreq400Hz

###### FirmwareMode

Information about the current mode of operation of the device firmware. It can be in two states:

- ModeBootloader - the device is in bootloader mode
- ModeApplication - normal operation

###### Java
```java
SensorFirmwareMode fwMode = sensor.getFirmwareMode();

// only for Callibri/ Kolibri
sensor.setFirmwareMode(SensorFirmwareMode.ModeApplication);
```

###### Kotlin
```kotlin
val fwMode: SensorFirmwareMode = sensor.firmwareMode

// only for Callibri/ Kolibri
sensor.firmwareMode = SensorFirmwareMode.ModeApplication
```

###### SamplingFrequency

An property that is used to set or get the sample rate of a physiological signal. The higher the value, the more data flow from the device to the application, which means the higher the power consumption, but also the higher the range of measured frequencies. And there are also limitations on the physical communication channel (BLE) in terms of bandwidth.

Recommendations for choosing a value:

- For EEG signals not less than 250 Hz;
- For ECG signals 125 Hz;
- For EMG not less than 1000 Hz. When working with several devices at the same time, it is not recommended to increase the frequency above 1000 Hz;
- The breath channel has a fixed sampling rate of 20 Hertz. MEMS channels have a fixed sampling rate of 100 Hertz.

It is unchanged for BrainBit and BrainBitBlack and is 250 Hz. Can be changed for Signal Callibri/Kolibri and can take on the following values:

- 125 Hz
- 250 Hz
- 500 Hz
- 1000 Hz
- 2000 Hz

Not available for Callibi/Kolibri.
If you try to set an unsupported value to the device, an exception will be thrown.

###### Java
```java
// get
SensorSamplingFrequency sf = sensor.getSamplingFrequency();

// set only for Callibri/Kolibri MF
sensor.setSamplingFrequency(SamplingFrequency.FrequencyHz1000);
```

###### Kotlin
```kotlin
var sf: SensorSamplingFrequency = sensor.samplingFrequency
// set only for Callibri/Kolibri MF
sensor.samplingFrequency = SamplingFrequency.FrequencyHz1000
```

###### Gain

Gain of an ADC signal. The higher the gain of the input signal, the less noise in the signal, but also the lower the maximum amplitude of the input signal. It is unchanged for BrainBit and BrainBitBlack and is 6. For signal Callibi/Kolibri you can set the desired value. Not available for Callibi/Kolibri EMS.

- 1
- 2
- 3
- 4
- 6
- 8
- 12
- 24

If you try to set an unsupported value to the device, an exception will be thrown.

###### Java
```java
SensorGain gain = sensor.getGain();
...
sensor.setGain(SensorGain.Gain6);
```

###### Kotlin
```kotlin
var gain: SensorGain = sensor.gain
...
sensor.gain = SensorGain.Gain6
```

###### Offset

Signal offset. It is unchanged for BrainBit and BrainBitBlack and is 0. For signal Callibi/Kolibri you can set the desired value. Not available for Callibi/Kolibri EMS.

- 1
- 2
- 3
- 4
- 5
- 6
- 7
- 8

If you try to set an unsupported value to the device, an exception will be thrown.

###### Java
```java
SensorDataOffset offset = sensor.getDataOffset();

// for Callibri/Kalibri MF
sensor.setDataOffset(SensorDataOffset.DataOffset3);
```

###### Kotlin
```kotlin
var offset: SensorDataOffset = sensor.dataOffset

// for Callibri/Kalibri MF
sensor.dataOffset = SensorDataOffset.DataOffset3
```

###### ExternalSwitchState

Switched signal source. This parameter is available only to Callibi/Kolibri. It is can take on the following values:

- ExtSwInMioElectrodesRespUSB - Respiratory channel data source is USB connector. The source of myographic channel data are terminals.
- ExtSwInMioElectrodes - Terminals are the source of myographic channel data. The breathing channel is not used.
- ExtSwInMioUSB - The source of myographic channel data is the USB connector. The breathing channel is not used.
- ExtSwInRespUSB - Respiratory channel data source is USB connector. Myographic channel is not used.

The value is stored and changed in the `ExtSwInput` property.

###### ADCInputState

State value of an ADC (Analog to Digital Converter) input of a device. This property is available only to Callibi/Kolibri. It is can take on the following values:

- Electrodes - Inputs to electrodes. This mode is designed to receive a physiological signal from the place of application.
- Short - Inputs are short-circuited. This mode is designed to close the outputs. The output will have noise at the baseline or 0 volt level.
- Test - Inputs to the ADC test. This mode is intended for checking the correctness of the ADC operation. The output should be a square wave signal with a frequency of 1 Hz and an amplitude of +/- 1 mV.
- Resistance - Inputs for measuring the interelectrode resistance. This mode is designed to measure the interelectrode resistance, as well as to obtain a physiological signal. This is the recommended default mode.

The value is stored and changed in the `ADCInput` property.

###### AccelerometerSens

The sensitivity value of the accelerometer, if the device supports it. This property is available only to Callibi/Kolibri. It is recommended to check the presence of the MEMS module before use. It is can take on the following values:

- 2g - Normal sensitivity. Minimum value. Sufficient for practical use
- 4g - Increased sensitivity.
- 8g - High sensitivity.
- 16g - Maximum sensitivity.

The value is stored and changed in the `AccSens` property.

###### GyroscopeSens

The gyroscope gain value, if the device supports it. This property is available only to Callibi/Kolibri. It is recommended to check the presence of the MEMS module before use. It is can take on the following values:

- 250Grad - The range of measured values of the angular velocity is from 0 to 2000 degrees per second. Recommended for measuring angles.
- 500Grad - The range of measured values of the angular velocity is from 0 to 1000 degrees per second.
- 1000Grad - The range of measured values of the angular velocity is from 0 to 500 degrees per second.
- 2000Grad - The range of measured values of the angular velocity is from 0 to 250 degrees per second.

The value is stored and changed in the `GyroSens` property.

###### StimulatorAndMAState

Parameter for obtaining information about the state of the stimulation mode and the motion assistant mode. This parameter is available only to Callibi/Kolibri EMS. Contains:

- StimulatorState - Stimulation mode state
- MAState - Drive assistant mode state

Each of the fields can be in three states:
 - Enabled
 - Disabled
 - NoParams
 
###### StimulatorParamPack

Stimulation parameters. This property is available only to Callibi/Kolibri EMS. Contains:

- Current - stimulus amplitude in  mA. 1..100.
- PulseWidth - duration of the stimulating pulse by us. 20..460.
- Frequency - frequency of stimulation impulses by Hz. 1..200.
- StimulusDuration - maximum stimulation time by ms. 0...65535.

The value is stored and changed in the `StimulatorParamCallibri` property.

###### MotionAssistantParamPack

Parameter for describing a stimulation mode, if the device supports this mode. This structure describes the parameters for starting the stimulation mode depending on the place of application of the device, the position of the limb in space and the pause between the starts of this mode while the specified conditions are met. Параметр доступен только для Callibi/Kolibri стимулятора. Contain a structure named `CallibriMotionAssistantParams` with fields:

- GyroStart - Angle value in degrees at which the stimulation mode will start, if it is correctly configured.
- GyroStop - Angle value in degrees above which the stimulation mode will stop, if it is correctly configured.
- Limb - multiple of 10. This means that the device is using the (MinPauseMs / 10) value. Correct values: 10, 20, 30, 40 ... 
- MinPauseMs - Pause between starts of stimulation mode in milliseconds. 

The value is stored and changed in the `MotionAssistantParamCallibri` property.

###### FirmwareVersion

Information about the device firmware version. Contain a structure named `SensorVersion` with fields:

- firmware major
- firmware minor
- firmware patch
- hardware major
- hardware minor
- hardware patch
- extension major

The value is stored and changed in the `Version` property.

###### MEMSCalibrationStatus

Calibration status of MEMS sensors. Conditional type.

###### MotionCounterParamPack

This parameter is available only to Callibi/Kolibri. Contain a structure named `MotionCounterParamCallibri` with fields:

- InsenseThresholdMG - Insense threshold mg. 0..500
- InsenseThresholdSample - Algorithm insense threshold in time (in samples with the MEMS sampling rate) 0..500

The value is stored and changed in the `CallibriMotionCounterParam` property.

###### MotionCounter

Contains the number of motions. This parameter is available only to Callibi/Kolibri. A numeric value that cannot be changed. The value is stored and changed in the `MotionCounterCallibri` property.

###### BattPower

Battery power value. Integer value.

###### SensorFamily

Type of device. Enumeration.

###### SensorMode

Operating mode of the physiological amplifier. The list of modes is different for BrainBit/BrainBitBlack and Callibri/Kolibri. The value is stored in the `SensorAmpMode` property.

###### SamplingFrequencyResist

Frequency of updating resistance values. Immutable value. Not available for Callibri/Kolibri. Don't has a fixed value for BrainBit/BrainBitBlack.

###### SamplingFrequencyMEMS

Frequency of updating MEMS values. Immutable value. Available for Callibri/Kolibri supporting MEMS.

###### SamplingFrequencyResp

Frequency of updating breath values. Неизмениемое значение. Available for Callibri/Kolibri supporting breath.

##### Features

Each device has a specific set of modules. You can find out which modules the device has using the property `Feature`:

###### Java

```java
List<SensorFeature> features = sensor.getSupportedFeature();
features.forEach(feature -> Log.i("TAG", feature.toString()));

// Output:
//
// Signal
// Resistance
```

###### Kotlin

```kotlin
val features = sensor.supportedFeatures
features!!.forEach { feature: SensorFeature ->
    Log.i(
        "TAG",
        feature.toString()
    )
}

// Output:
//
// Signal
// Resistance
```

You can also check if the feature is supported, for example `Signal`:

###### Java

```java
if(sensor.isSupportedFeature(SensorFeature.Signal)){
...
}
```

###### Kotlin

```kotlin
if(sensor.isSupportedFeature(SensorFeature.Signal) == true){
...
}
```

##### Commands

The device can execute certain commands. The list of supported commands can be obtained as follows:

###### Java

```java
List<SensorCommand> commands = sensor.getCommands();
commands.forEach(command -> Log.i("TAG", command.toString()));

// Output:
//
// StartSignal
// StopSignal
// StartResist
// StopResist
```

###### Kotlin

```kotlin
val commands = sensor.commands
commands!!.forEach { command: SensorCommand ->
    Log.i(
        "TAG",
        command.toString()
    )
}

// Output:
//
// StartSignal
// StopSignal
// StartResist
// StopResist
```

And also check if the device can execute the desired command:

###### Java

```java
if(sensor.isSupportedCommand(SensorCommand.StartSignal)){
...
}
```

###### Kotlin

```kotlin
if(sensor.isSupportedCommand(SensorCommand.StartSignal) == true){
...
}
```


### BrainBit, BrainBitBlack

The BrainBit and BrainBitBlack is a headband with 4 electrodes and 4 data channels - O1, O2, T3, T4. The device has a frequency of 250 Hz, which means that data on each of the channels will come at a frequency of 250 samples per second. The parameters of this device, such as gain, data offset and the other, cannot be changed, if you try to do this, an exception will appear.

###### Kotlin
```csharp
sensor.gain = SensorGain.SensorGain1; // <- This throw an exeption!
```

>  You can distinguish BrainBit device from Flex by the firmware version number: if the `SensorVersion.FwMajor` is more than 100 - it's Flex, if it's less than BrainBit.

> BrainBitBlack, unlike BrainBit, requires pairing with a PC/mobile device. So, before connecting to the BBB, you must put it into pairing mode. SDK starts the pairing process automatically. 



#### Receiving signal

To receive signal data, you need to subscribe to the corresponding callback. The values will be received as a packet from four channels at once, which will avoid desynchronization between them. The values come in volts. In order for the device to start transmitting data, you need to start a signal using the `execute` command. This method is also recommended to be run in an separate thread.

###### Java

```java
sensor.brainBitSignalDataReceived = data -> {
    Log.i("TAG", Arrays.toString(data));
};
sensor.ExecCommand(SensorCommand.StartSignal);
...
sensor.brainBitSignalDataReceived = null
sensor.ExecCommand(SensorCommand.StopSignal);
```

###### Kotlin

```kotlin
sensor.brainBitSignalDataReceived = Sensor.BrainBitSignalDataReceived { data ->
    Log.i("TAG", data.toString())
}
sensor.ExecCommand(SensorCommand.StartSignal)
...
sensor.brainBitSignalDataReceived = null
sensor.ExecCommand(SensorCommand.StopSignal)
```

You get signal values as a list of samples, each containing:
 - PackNum -  number for each packet
 - Marker - marker of sample, if it was sent and this feature is supported by the device
 - O1 - value of O1 channel in V
 - O2 - value of O2 channel in V
 - T3 - value of T3 channel in V
 - T4 - value of T4 channel in V

#### Ping signal

Some devices support signal quality check functions using signal ping. You can send a specific value (marker) to the device and it will return that marker with the next signal data packet. Marker is small value one byte in size.

> Available only to BrainBitBlack

###### Java

```java
sensor.pingNeuroSmart(5);
```

###### Kotlin

```kotlin
sensor.pingNeuroSmart(5)
```

#### Recieving resistance

BrainBit and BrainBitBlack also allow you to get resistance values. With their help, you can determine the quality of the electrodes to the skin. Initial resistance values are infinity. The values change when the BB is on the head.

For BrainBit the upper limit of resistance is 2.5 ohms.

###### Java

```java
sensor.brainBitResistDataReceived = data -> {
    Log.i("TAG", Arrays.toString(data));
};
sensor.ExecCommand(SensorCommand.StartResist);
...
sensor.brainBitResistDataReceived = null
sensor.ExecCommand(SensorCommand.StopResist);
```

###### Kotlin

```kotlin
sensor.brainBitResistDataReceived = Sensor.BrainBitResistDataReceived { data ->
    Log.i("TAG", data.toString())
}
sensor.ExecCommand(SensorCommand.StartResist)
...
sensor.brainBitResistDataReceived = null
sensor.ExecCommand(SensorCommand.StopResist)
```

You get resistance values structure of samples for each channel:
 - O1 - value of O1 channel in Ohm
 - O2 - value of O2 channel in Ohm
 - T3 - value of T3 channel in Ohm
 - T4 - value of T4 channel in Ohm

### BrainBit 2/Flex/Pro

The BrainBit2 class is designed to work with several device families: BrainBit 2, BrainBit Pro, BrainBit Flex, BrainBit Flex Pro. All devices have a sampling frequency of 250Hz. All devices can work in two modes - signal and resistance separately. These devices have different number of channels - BrainBit2, BrainBit Flex have 4 channels each and BrainBitPro, BrainBit FlexPro have 8 channels each. The main difference from BraibBit of the first version is that they do not support gain property, but have the ability to set gain for each channel separately using `BrainBit2AmplifierParam` structure. Also, these types of devices support the [ping command](#ping-signal).

#### Info about channels

The device can have 4 or 8 channels. The number of channels can be determined as follows:

###### Java
```java
List<EEGChannelInfo> channels = getSupportedChannels();
```

###### Kotlin
```kotlin
var channels: List<EEGChannelInfo> = sens.supportedChannels
```

`EEGChannelInfo` contains some info:

###### Java
| Field | Type | Description |
|--|--|--|
|Id|EEGChannelId|physical location of the channel. You will receive the values `O1`, `O2`, `T3`, `T4` or `Unknown`. `Unknown` means that the position of a specific electrode is free.|
|ChType|EEGChannelType|type of channel, possible values `A1`, `A2`, `Differential` or `Referent`|
|Name|String|channel name|
|Num|short|channel number. By this number the channel will be located in the array of signal or resistance values|

###### Kotlin
| Field | Type | Description |
|--|--|--|
|Id|EEGChannelId!|physical location of the channel. You will receive the values `O1`, `O2`, `T3`, `T4` or `Unknown`. `Unknown` means that the position of a specific electrode is free.|
|ChType|EEGChannelType!|type of channel, possible values `A1`, `A2`, `Differential` or `Referent`|
|Name|String!|channel name|
|Num|Short|channel number. By this number the channel will be located in the array of signal or resistance values|

Also you can check only channels count without info:

###### Java
```java
int chCount = sensor.getChannelsCount();
```

###### Kotlin
```kotlin
val chCount: Int = sensor.channelsCount
```

#### AmpMode

This device can show it's current amplifier mode. It can be in the following states:

  - Invalid
  - PowerDown
  - Idle
  - Signal
  - Resist

You can check amp. mode by two ways:

1. by callback:

###### Java
```java
sensor.sensorAmpModeChanged = mode -> {
    // check AmpMode
};
...
sensor.sensorAmpModeChanged = null;
```

###### Kotlin
```kotlin
sensor.sensorAmpModeChanged = SensorAmpModeChanged {mode -> 
    // check AmpMode    
}
...
sensor.sensorAmpModeChanged = null
```

2. get value at any time:

###### Java
```java
SensorAmpMode mode = sensor.getAmpMode();
```

###### Kotlin
```kotlin
var mode: SensorAmpMode = sensor.ampMode
```

It is very important parameter for BrainBit2 device because you can set amplifier parameters only if device into `PowerDown` or `Idle` mode.

#### Amplifier parameters

You can configure each channel and whole device settings by setting amplifier parameters.

###### Java
```java
BrainBit2AmplifierParam ampParam = sensor.getAmplifierParam();
for (int i = 0; i < sensor.getChannelsCount(); ++i) {
    ampParam.getChSignalMode()[i] = BrainBit2ChannelMode.ChModeNormal;
    ampParam.getChGain()[i] = SensorGain.Gain6;
    ampParam.getChResistUse()[i] = true;
}
ampParam.setCurrent(GenCurrent.GenCurr6nA);
sensor.setAmplifierParam(ampParam);
```

###### Kotlin
```kotlin
val ampParam: BrainBit2AmplifierParam = sensor.amplifierParam
for (i in 0 until sensor!!.channelsCount) {
    ampParam.chSignalMode[i] = BrainBit2ChannelMode.ChModeNormal
    ampParam.chGain[i] = SensorGain.Gain6
    ampParam.chResistUse[i] = true
}
ampParam.current = GenCurrent.GenCurr6nA
sensor.amplifierParam = ampParam
```

`BrainBit2AmplifierParam` contains:

###### Java
| Field | Type | Description |
|--|--|--|
|ChSignalMode|BrainBit2ChannelMode[]|input type|
|ChResistUse|boolean[]|is use for resistance|
|ChGain|SensorGain[]|gain of an ADC signal for each channel|
|Current|GenCurrent|setting parameters of the probe current generator|

###### Kotlin
| Field | Type | Description |
|--|--|--|
|ChSignalMode|Array<BrainBit2ChannelMode>|input type|
|ChResistUse|BooleanArray|is use for resistance|
|ChGain|Array<SensorGain>|gain of an ADC signal for each channel|
|Current|GenCurrent!|setting parameters of the probe current generator|

Possible values for `Current`:
 - 0nA 
 - 6nA 
 - 12nA
 - 18nA
 - 24nA

Signal modes:
 - Short - shorted input
 - Normal - bipolar input mode (used for EEG)

Possible `Gain` values:
 - 1
 - 2
 - 3
 - 4
 - 6
 - 8
 - 12

#### Receiving signal

To receive signal data, you need to subscribe to the corresponding callback. The values come in volts. In order for the device to start transmitting data, you need to start a signal using the `execute` command. This method is also recommended to be run in an separate thread.

###### Java
```java
sensor.signalDataReceived = data ->{
    // process data
};
sensor.execCommand(SensorCommand.StartSignal);
...
sensor.signalDataReceived = null
sensor.execCommand(SensorCommand.StopSignal);
```

###### Kotlin
```kotlin
sensor.signalDataReceived = BrainBit2SignalDataReceived { data ->
    // process data
}
sensor.execCommand(SensorCommand.StartSignal)
...
sensor.signalDataReceived = null
sensor.execCommand(SensorCommand.StopSignal)
```

You get signal values as a list of samples (`SignalChannelsData`), each containing:

###### Java
| Field | Type | Description |
|--|--|--|
|PackNum|int|number for each packet|
|Marker|short|marker of sample|
|Samples|double[]|array of samples in V. Each sample number into array consistent with `Num` value of [EEGChannelInfo](#info-about-channels) from `getSupportedChannels()` method.|

###### Kotlin
| Field | Type | Description |
|--|--|--|
|PackNum|Int|number for each packet|
|Marker|Short|marker of sample|
|Samples|DoubleArray|array of samples in V. Each sample number into array consistent with `Num` value of [EEGChannelInfo](#info-about-channels) from `getSupportedChannels()` method.|

#### Receiving resistance

BrainBit2 also allow you to get resistance values. With their help, you can determine the quality of the electrodes to the skin. Initial resistance values are infinity. The values change when the device is on the head.

###### Java
```java
sensor.resistDataReceived = data ->{
    // process data
};
sensor.execCommand(SensorCommand.StartResist);
...
sensor.resistDataReceived = null
sensor.execCommand(SensorCommand.StopResist);
```

###### Kotlin
```kotlin
sensor.resistDataReceived = BrainBit2ResistDataReceived { data ->
    // process data
}
sensor.execCommand(SensorCommand.StartResist)
...
sensor.resistDataReceived = null
sensor.execCommand(SensorCommand.StopResist)
```

You get resistance values structure of samples (`ResistRefChannelsData`) for each channel:

###### Java
| Field | Type | Description |
|--|--|--|
|PackNum|int|number for each packet|
|Samples|double[]|array of samples in V. Each sample number into array consistent with `Num` value of [EEGChannelInfo](#info-about-channels) from `getSupportedChannels()` method.|
|Referents|double[]|array of values for referents channels. For BrainBit2 sensor is empty now.|

###### Kotlin
| Field | Type | Description |
|--|--|--|
|PackNum|Int|number for each packet|
|Samples|DoubleArray|array of samples in V. Each sample number into array consistent with `Num` value of [EEGChannelInfo](#info-about-channels) from `getSupportedChannels()` method.|
|Referents|DoubleArray|array of values for referents channels. For BrainBit2 sensor is empty now.|

### Callibri MF/ Kolibri MF

The Callibri family of devices has a wide range of built-in modules. For each of these modules, the SDK contains its own processing area. It is recommended before using any of the modules to check if the module is supported by the device using one of the methods `IsSupportedFeature`, `IsSupportedCommand` or `IsSupportedParameter`

#### Receiving signal

To receive signal data, you need to subscribe to the corresponding callback. The values come in volts. In order for the device to start transmitting data, you need to start a signal using the `execute` command. This method is also recommended to be run in an separate thread.

The sampling rate can be controlled using the `SamplingFrequency` property. For example, at a frequency of 1000 Hz, the device will send 1000 samples per second. Supports frequencies 125/250/500/1000/2000 Hz. You can also adjust the signal offset (`DataOffset`) and signal power (`Gain`).

###### Java

```java
sensor.callibriSignalDataReceived = data -> {
    Log.i("TAG", Arrays.toString(data));
};
sensor.ExecCommand(SensorCommand.StartSignal);
...
sensor.callibriSignalDataReceived = null
sensor.ExecCommand(SensorCommand.StopSignal);
```

###### Kotlin

```kotlin
sensor.callibriSignalDataReceived = Sensor.CallibriSignalDataReceived { data ->
    Log.i("TAG", data.toString())
}
sensor.ExecCommand(SensorCommand.StartSignal)
...
sensor.callibriSignalDataReceived = null
sensor.ExecCommand(SensorCommand.StopSignal)
```

You get signal values as a list of samples, each containing:
 - PackNum -  number for each packet
 - array of samples in V


#### Signal settings

By default, the Callibri/Kolibri gives a signal without filters. In order to receive a certain type of signal, for example, EEG or ECG, you need to configure the device in a certain way. For this there is a property `SignalTypeCallibri`. Preset signal types include:
 - EEG - parameters: Gain6, Offset = 3,  ADCInputResistance
 - EMG - parameters: Gain6, Offset = 3,  ADCInputResistance
 - ECG - parameters: Gain6, Offset = 3,  ADCInputResistance
 - EDA (GSR) - parameters: Gain6, Offset = 8,  ADCInputResistance, ExternalSwitchInputMioElectrodes. By default the input to the terminals is set. If you want to change it to USB use the `ExtSwInput` property.
 - StrainGaugeBreathing - parameters: Gain6, Offset = 4,  ADCInputResistance, ExternalSwitchInputMioUSB
 - ImpedanceBreathing - parameters: Gain6, Offset = 4,  ADCInputResistance, ExternalSwitchInputRespUSB

Hardware filters disabled by default for all signal types. You can enable filters by `HardwareFilters` property, for example LP filter. 

> Important!
> When using an LP filter in the sensor, you will not see the constant component of the signal.


#### Receiving envelope

To get the values of the envelope, you need to subscribe to a specific event and start pickup. The channel must be configured in the same way as for a normal signal, and all parameters work the same way. Then the signal is filtered and decimated at 20 Hz.

###### Java

```java
sensor.callibriEnvelopeDataReceived = data -> {
    Log.i("TAG", Arrays.toString(data));
};
sensor.ExecCommand(SensorCommand.StartEnvelope);
...
sensor.callibriEnvelopeDataReceived = null
sensor.ExecCommand(SensorCommand.StopEnvelope);
```

###### Kotlin

```kotlin
sensor.callibriEnvelopeDataReceived = Sensor.CallibriSignalDataReceived { data ->
    Log.i("TAG", data.toString())
}
sensor.ExecCommand(SensorCommand.StartEnvelope)
...
sensor.callibriEnvelopeDataReceived = null
sensor.ExecCommand(SensorCommand.StopEnvelope)
```

You get signal values as a list of samples, each containing:
 - PackNum - number for each packet
 - sample in V

#### Check electrodes state

Allows you to determine the presence of electrical contact of the device electrodes with human skin. It can be in three states:
 - Normal - The electrodes have normal skin contact. Low electrical resistance between electrodes. The expected state when working with physiological signals.
 - Detached - High electrical resistance between electrodes. High probability of interference in the physiological signal.
 - HighResistance - There is no electrical contact between the electrodes of the device.
 
To receive data, you need to subscribe to the corresponding callback and start signal pickup.

###### Java

```java
sensor.callibriElectrodeStateChanged = elState -> {
    Log.i("TAG", elState.toString());
};
sensor.ExecCommand(SensorCommand.StartSignal);
...
sensor.callibriElectrodeStateChanged = null
sensor.ExecCommand(SensorCommand.StopSignal);
```

###### Kotlin

```kotlin
sensor.callibriElectrodeStateChanged = Sensor.CallibriSignalDataReceived { elState ->
    Log.i("TAG", elState.toString())
}
sensor.ExecCommand(SensorCommand.StartSignal)
...
sensor.callibriElectrodeStateChanged = null
sensor.ExecCommand(SensorCommand.StopSignal)
```

You get signal values as a list of samples, each containing:
 - PackNum - number for each packet
 - sample in V

#### Receiving Respiration

The breathing microcircuit is optional on request. Its presence can be checked using the `IsSupportedFeature` method. To receive data, you need to connect to the device, subscribe to the notification of data receipt and start picking up.

###### Java

```java
if(sensor.isSupportedFeature(SensorFeature.Respiration))
{
    sensor.callibriRespirationDataReceived = data -> {
        Log.i("TAG", Arrays.toString(data));
    };
    sensor.ExecCommand(SensorCommand.StartRespiration);
    ...
    sensor.callibriRespirationDataReceived = null
    sensor.ExecCommand(SensorCommand.StopRespiration);
}
```

###### Kotlin

```kotlin
if(sensor.isSupportedFeature(SensorFeature.Respiration) == true)
{
    sensor.callibriRespirationDataReceived = Sensor.CallibriRespirationDataReceived { data ->
        Log.i("TAG", data.toString())
    }
    sensor.ExecCommand(SensorCommand.StartRespiration)
    ...
    sensor.callibriRespirationDataReceived = null
    sensor.ExecCommand(SensorCommand.StopRespiration)
}
```

You get signal values as a list of samples, each containing:
 - PackNum - number for each packet
 - array of samples in V

#### MEMS

The MEMS microcircuit is optional on request. Its presence can be checked using the `IsSupportedFeature` method. This means that the device contains an accelerometer and a gyroscope. Contains information about the position of the device in space. Channel sampling frequency is 100 Hz.

MEMS data is a structure:

- PackNum - number for each packet
- Accelerometer - accelerometer data. Contains:
  - X - Abscissa Acceleration
  - Y - Y-axis acceleration
  - Z - Acceleration along the applicate axis
- Gyroscope - gyroscope data
  - X - The angle of inclination along the abscissa axis
  - Y - Inclination angle along the ordinate axis
  - Z - Angle of inclination along the axis of the applicate

Quaternion data is a structure:

- PackNum - number for each packet
- W - Rotation component
- X - Vector abscissa coordinate
- Y - Vector coordinate along the ordinate axis
- Z - The coordinate of the vector along the axis of the applicate

It is recommended to perform calibration on a flat, horizontal non-vibrating surface before starting work using the `CalibrateMEMS` command. Calibration state can be checked using the `MEMSCalibrateStateCallibri` property, it can take only two values: calibrated (true), not calibrated (false).

> MEMS and quaternion available only to signal Callibri/Kolibri!

###### Java

```java
// Calibration
sensor.ExecCommand(SensorCommand.CalibrateMEMS);
boolean isCalibrated = sensor.getMEMSCalibrateStateCallibri();

// For receiving MEMS
sensor.memsDataReceived = memsData -> {
    Log.i("TAG", Arrays.toString(memsData));
};
sensor.ExecCommand(SensorCommand.StartMEMS);
...
sensor.memsDataReceived = null;
sensor.ExecCommand(SensorCommand.StopMEMS);

// For quaternion
sensor.quaternionDataRecieved = quaternionData -> {
    Log.i("TAG", Arrays.toString(quaternionData));
};
sensor.ExecCommand(SensorCommand.StartAngle);
...
sensor.quaternionDataRecieved = null;
sensor.ExecCommand(SensorCommand.StopAngle);
```

###### Kotlin

```kotlin
// Calibration
sensor.ExecCommand(SensorCommand.CalibrateMEMS)
var isCalibrated: Boolean = sensor.getMEMSCalibrateStateCallibri() == true

// For receiving MEMS
sensor.memsDataReceived = Sensor.MEMSDataReceived -> {
    Log.i("TAG", Arrays.toString(it));
};
sensor.ExecCommand(SensorCommand.StartMEMS)
...
sensor.memsDataReceived = null
sensor.ExecCommand(SensorCommand.StopMEMS)

// For quaternion
sensor.quaternionDataRecieved = Sensor.QuaternionDataReceived -> {
    Log.i("TAG", Arrays.toString(it));
};
sensor.ExecCommand(SensorCommand.StartAngle)
...
sensor.quaternionDataRecieved = null
sensor.ExecCommand(SensorCommand.StopAngle)
```

#### Motion counter

Represents a motion counter. It can be configured using the `CallibriMotionCounterParam` property, in it:

 - InsensThreshmG – Threshold of the algorithm's deadness in amplitude (in mg). The maximum value is 500mg. The minimum value is 0.
 - InsensThreshSamp - Threshold of the algorithm's insensitivity in time (in samples with the MEMS sampling rate). The maximum value is 500 samples. The minimum value is 0.

You can find out the current number of movements using the `MotionCounterCallibri` property. You can reset the counter with the `ResetMotionCounter` command. No additional commands are needed to start the counter, it will be incremented all the time until the reset command is executed.

###### Java

```java
if(sensor.isSupportedParameter(SensorParameter.ParameterMotionCounter))
{
    sensor.setMotionCounterParamCallibri(new CallibriMotionCounterParam((short)250, (short)250));
    int motionCount = sensor.getMotionCounterCallibri();
    sensor.ExecCommand(SensorCommand.ResetMotionCounter);
}
```

###### Kotlin

```kotlin
if(sensor.isSupportedParameter(SensorParameter.ParameterMotionCounter) == true)
{
    sensor.motionCounterParamCallibri = CallibriMotionCounterParam(
                250.toShort(),
                250.toShort()
            )
    val motionCount: Int = sensor.motionCounterCallibri
    sensor.ExecCommand(SensorCommand.ResetMotionCounter)
}
```

### Callibri/Kolibri EMS

Kallibri is a EMS if it supports the stimulation module:

###### Java

```java
boolean isStimulator = sensor.isSupportedFeature(SensorFeature.CurrentStimulator);
```

###### Kotlin

```kotlin
var isSupported: Boolean = sensor.isSupportedFeature(SensorFeature.CurrentStimulator) == true
```

#### Stimulation

Before starting the session, you need to correctly configure the device, otherwise the current strength may be too strong or the duration of stimulation too long. The setting is done using the `StimulatorParamCallibri` property. You can set the following options:

 - Current - stimulus amplitude in  mA. 1..100
 - PulseWidth - duration of the stimulating pulse by us. 20..460
 - Frequency - frequency of stimulation impulses by Hz. 1..200.
 - StimulusDuration - maximum stimulation time by ms. 0...65535. Zero is infinitely.

You can start and stop stimulation with the following commands:

###### Java

```java
sensor.execCommand(SensorCommand.StartCurrentStimulation);
...
sensor.execCommand(SensorCommand.StopCurrentStimulation);
```

###### Kotlin

```kotlin
sensor.execCommand(SensorCommand.StartCurrentStimulation)
...
sensor.execCommand(SensorCommand.StopCurrentStimulation)
```

> Stimulation does not stop after the `StimulusDuration` time has elapsed.

You can check the state of stimulation using the `StimulatorMAStateCallibri` property. Contains two parameters:

- StimulatorState - Stimulation mode state
- MAState - Drive assistant mode state

Each of the parameters can be in 4 states:

- StimStateNoParams - parameter not set
- StimStateDisabled - mode disabled
- StimStateEnabled - mode enabled
- StimStateUnsupported - sensor unsupported

#### Motion assistant

The Callibri EMS, which contains the MEMS module, can act as a motion corrector. You can set the initial and final angle of the device and the limb on which the Callibri/Kolibri is installed, as well as a pause between stimulations and turn on the motion assistant. All the time while the device is tilted in the set range, stimulation will met. Stimulation will take place according to the settings of `StimulatorParamCallibri`.

The motion corrector works in the background. After turning on the motion assistant mode, it will work regardless of the connection to a mobile device/PC. You can turn on the motion corrector mode using a special command. When the device is rebooted, it is also reset.

Motion corrector parameters are a structure with fields:

 - GyroStart - Angle value in degrees at which the stimulation mode will start, if it is correctly configured.
 - GyroStop - Angle value in degrees above which the stimulation mode will stop, if it is correctly configured.
 - Limb - overlay location in stimulation mode, if supported.
 - MinPauseMs - Pause between starts of stimulation mode in milliseconds. Multiple of 10. This means that the device is using the (MinPauseMs / 10) value. Correct values: 10, 20, 30, 40 ... 

###### Java

```java
sensor.setMotionAssistantParamCallibri(new CallibriMotionAssistantParams(
        (byte)45, 
        (byte)10, 
        CallibriMotionAssistantLimb.RightLeg.index(), 
        (byte)10)
    );
sensor.ExecCommand(SensorCommand.EnableMotionAssistant);
...
sensor.ExecCommand(SensorCommand.DisableMotionAssistant);
```

###### Kotlin

```kotlin
sensor.motionAssistantParamCallibri = CallibriMotionAssistantParams(
                45.toByte(),
                10.toByte(),
                CallibriMotionAssistantLimb.RightLeg.index(),
                10.toByte()
            )
sensor.ExecCommand(SensorCommand.EnableMotionAssistant)
...
sensor.ExecCommand(SensorCommand.DisableMotionAssistant)
```

## NeuroEEG (DragonEEG)

DragonEEG requires pairing with a PC/mobile device. So, before connecting to device, you must put it into pairing mode. SDK starts the pairing process automatically.

Allows for long-term monitoring of brain biopotentials through 21 channels, with parallel registration of three polygraphic channels: ECG, EMG and EOG.

DragonEEG device supports the next signal frequencies:
 - 1000 Hz
 - 500 Hz
 - 250 Hz

And gain values:
 - 1
 - 2
 - 4
 - 6
 - 8
 - 12
 - 24

### Info about channels

NeuroEEG device has 24 channels. You can get channels count by `channelsCount` property. Get channels count:

###### Java
```java
int chCount = sensor.getChannelsCount();
```

###### Kotlin
```kotlin
val chCount: Int = sensor.channelsCount
```

Receive supportes channels info:

###### Java
```java
List<EEGChannelInfo> channels = sensor.getSupportedChannels();
```

###### Kotlin
```kotlin
var channels: List<EEGChannelInfo> = sens.supportedChannels
```

`EEGChannelInfo` contains some info:
 - Id - `EEGChannelId` type - physical location of the channel. Possible values:
  - O1 
  - P3 
  - C3 
  - F3 
  - Fp1
  - T5 
  - T3 
  - F7 
  - F8 
  - T4 
  - T6 
  - Fp2
  - F4 
  - C4 
  - P4 
  - O2 
  - D1 
  - D2 
  - OZ 
  - PZ 
  - CZ 
  - FZ 
  - FpZ
  - D3 
 - ChType - `EEGChannelType` type - type of channel, possible values A1, A2, differential or referent
 - Name - `string` type - channel name
 - Num - `int` type - channel number. By this number the channel will be located in the array of signal or resistance values

### AmpMode

This device can show it's current amplifier mode. It can be in the following states:

  - Invalid
  - PowerDown
  - Idle
  - Signal
  - Resist
  - SignalResist

You can check amp. mode by two ways:

1. by callback:

###### Java
```java
sensor.sensorAmpModeChanged = mode -> {
    // check AmpMode
};
...
sensor.sensorAmpModeChanged = null;
```

###### Kotlin
```kotlin
sensor.sensorAmpModeChanged = SensorAmpModeChanged {mode -> 
    // check AmpMode    
}
...
sensor.sensorAmpModeChanged = null
```

2. get value at any time:

###### Java
```java
SensorAmpMode mode = sensor.getAmpMode();
```

###### Kotlin
```kotlin
var mode: SensorAmpMode = sensor.ampMode
```

It is very important parameter for NeuroEEG-M device because you can set amplifier parameters only if device into `PowerDown` or `Idle` mode.

### Amplifier params

After connecting the device, it is absolutely necessary to configure the parameters of the amplifier. If this is not done, the signal and resistance values will be 0.0 or will not appear at all.

###### Java
```java
NeuroEEGAmplifierParam ampParam = sensor.getAmplifierParam();
ampParam.setFrequency(SensorSamplingFrequency.FrequencyHz500);
ampParam.setReferentResistMesureAllow(true);
for (int i = 0; i < sensor.getChannelsCount(); ++i) {
    ampParam.ChannelMode[i] = EEGChannelMode.SignalResist;
    ampParam.ChannelGain[i] = SensorGain.Gain6;
}
sensor.setAmplifierParam(ampParam);
```

###### Kotlin
```kotlin
val ampParam: NeuroEEGAmplifierParam = sensor.amplifierParam
ampParam.frequency = SensorSamplingFrequency.FrequencyHz500
ampParam.referentResistMesureAllow = true
for (i in 0 until sensor!!.channelsCount) {
    ampParam.ChannelMode[i] = EEGChannelMode.SignalResist
    ampParam.ChannelGain[i] = SensorGain.Gain6
}
sensor.amplifierParam = ampParam
```

Channel modes:
 - SignalResist - device will be sent signal and resist values simultaneously
 - Signal - device will be sent only signal values, channel resistance will not be measured even if the resistance measurement mode is started in the device
 - Shorted - channel is shorted to ground, this mode can be used to analyze intrinsic noise of the device
 - Test - this channel will receive a test signal
 - Off - channel is disabled, no data from it

Referent modes:
 - A1A2 - both A1 and A2 references are available
 - HeadTop - only one reference is available, the resistance data is placed in field A1

ReferentResistMesureAllow - flag, allows resistance measurement by references (if true, the signal is not valid)

### Receiving signal

To receive signal data, you need to subscribe to the corresponding callback. The values come in volts. In order for the device to start transmitting data, you need to start a signal using the `execute` command. This method is also recommended to be run in an separate thread.

The sampling rate can be controlled using the `Frequency` value of amplifier parameters. For example, at a frequency of 1000 Hz, the device will send about 1000 samples per second. Supports frequencies 250/500/1000 Hz. You can also adjust signal power (`Gain`) value for each channel.

###### Java
```java
sensor.signalDataReceived = data -> {
    Log.i("TAG", Arrays.toString(data));
};
sensor.ExecCommand(SensorCommand.StartSignal);
...
sensor.signalDataReceived = null
sensor.ExecCommand(SensorCommand.StopSignal);
```

###### Kotlin
```kotlin
sensor.signalDataReceived = Sensor.NeuroEEGSignalDataReceived { data ->
    Log.i("TAG", data.toString())
}
sensor.ExecCommand(SensorCommand.StartSignal)
...
sensor.signalDataReceived = null
sensor.ExecCommand(SensorCommand.StopSignal)
```

You get signal values as a list of samples, each containing:
 - PackNum - number for each packet
 - Marker - marker of sample
 - Samples - array of samples in V. Each sample number into array consistent with `Num` value of `EEGChannelInfo` from `supported_channels` property.

### Receiving resistance

NeuroEEG also allow you to get resistance values. With their help, you can determine the quality of the electrodes to the skin. Initial resistance values are infinity. The values change when the device is on the head.

###### Java
```java
sensor.resistDataReceived = data -> {
    Log.i("TAG", Arrays.toString(data));
};
sensor.ExecCommand(SensorCommand.StartResist);
...
sensor.resistDataReceived = null
sensor.ExecCommand(SensorCommand.StopResist);
```

###### Kotlin
```kotlin
sensor.resistDataReceived = Sensor.NeuroEEGResistDataReceived { data ->
    Log.i("TAG", data.toString())
}
sensor.ExecCommand(SensorCommand.StartResist)
...
sensor.resistDataReceived = null
sensor.ExecCommand(SensorCommand.StopResist)
```

You get resistance values structure of samples for each channel:
 - PackNum - number for each packet
 - A1 - value of A1 channel in Ohm
 - A2 - value of A2 channel in Ohm
 - Bias - value of Bias channel in Ohm
 - Values - array of values in Ohm. Each sample number into array consistent with `Num` value of `EEGChannelInfo` from `getSupportedChannels()` method.

### Receiving signal and resistance

This device supports capturing signal and resistance at the same time. 

> NOTE:
> In this mode resistData may not arrive every update of the SignalResistReceived callback.

###### Java
```java
sensor.signalResistReceived = signalData, resistData -> {
    Log.i("TAG", Arrays.toString(signalData));
    Log.i("TAG", Arrays.toString(resistData));
};
sensor.ExecCommand(SensorCommand.StartSignalAndResist);
...
sensor.signalResistReceived = null
sensor.ExecCommand(SensorCommand.StopSignalAndResist);
```

###### Kotlin
```kotlin
sensor.signalResistReceived = Sensor.NeuroEEGSignalResistReceived { signalData, resistData ->
    Log.i("TAG", data.toString())
}
sensor.ExecCommand(SensorCommand.StartSignalAndResist)
...
sensor.signalResistReceived = null
sensor.ExecCommand(SensorCommand.StopSignalAndResist)
```