# nRF Connect Device Manager

This repository is a fork of the [McuManager Android Library](https://github.com/JuulLabs-OSS/mcumgr-android), 
which has been DEPRECATED. All new features and bug fixes will be added here. Please, migrate to the
new version to get future updates.

The sample application has been named nRF Connect Device Manager and is available on 
[Google Play](https://play.google.com/store/apps/details?id=no.nordicsemi.android.nrfconnectdevicemanager).

nRF Connect Device Manager library is compatible with Mcu Manager, a management subsystem supported 
by nRF Connect SDK, Zephyr and Apache Mynewt.

---

# McuManager Android

A transport agnostic implementation of the McuManager protocol. Contains a default implementation for BLE transport.
Minimum required Android version is 5.0 (Android Lollipop) due to a requirement for high MTU.

## Gradle Install

[ ![Download](https://maven-badges.herokuapp.com/maven-central/no.nordicsemi.android/mcumgr-ble/badge.svg?style=plastic) ](https://search.maven.org/search?q=g:no.nordicsemi.android)

#### McuManager BLE (Recommended)
Contains the core and a BLE transport implementation using Nordic's [Android-BLE-Library v2](https://github.com/NordicSemiconductor/Android-BLE-Library). 

```groovy
implementation 'no.nordicsemi.android:mcumgr-ble:0.13.0'
```

The core module will be included automatically.

#### McuManager Core
Core dependency only. Use if you want to provide your own transport implementation.

```groovy
implementation 'no.nordicsemi.android:mcumgr-code:0.13.0'
```

### Migration from the original repo

When migrating from the version 0.12 and older, change 
```groovy
implementation 'io.runtime.mcumgr:mcumgr-ble:0.XX.X'
```
to the above.

# Introduction

McuManager is an application layer protocol used to manage and monitor microcontrollers running 
Apache Mynewt and Zephyr. More specifically, McuManagr implements over-the-air (OTA) firmware upgrades, 
log and stat collection, and file-system and configuration management. 

## Command Groups

McuManager are organized by functionality into command groups. In _mcumgr-android_, command groups 
are called managers and extend the `McuManager` class. The managers (groups) implemented in 
_mcumgr-android_ are:

* **`DefaultManager`**: Contains commands relevant to the OS. This includes task and memory pool 
  statistics, device time read & write, and device reset.
* **`ImageManager`**: Manage image state on the device and perform image uploads.
* **`StatsManager`**: Read stats from the device.
* **`ConfigManager`**: Read/Write config values on the device.
* **`LogManager`**: Collect logs from the device.
* **`FsManager`**: Download/upload files from the device file system.

# Firmware Upgrade

Firmware upgrade is generally a four step process performed using commands from the `image` and 
`default` commands groups: `upload`, `test`, `reset`, and `confirm`.

This library provides a `FirmwareUpgradeManager` as a convenience for upgrading the image running on a device. 

### Example
```java
// Initialize the BLE transporter with context and a BluetoothDevice
McuMgrTransport transport = new McuMgrBleTransport(context, bluetoothDevice);

// Initialize the FirmwareUpgradeManager
FirmwareUpgradeManager dfuManager = new FirmwareUpgradeManager(transport, dfuCallback)

// Start the firmware upgrade with the image data
dfuManager.start(imageData);
```

## FirmwareUpgradeManager

A `FirmwareUpgradeManager` provides an easy way to perform firmware upgrades on a device. 
A `FirmwareUpgradeManager` must be initialized with an `McuMgrTransport` which defines the transport 
scheme and device. Once initialized, a `FirmwareUpgradeManager` can perform one firmware upgrade at a time. 
Firmware upgrades are started using the `start(byte[] imageData)` method and can be paused, resumed, 
and canceled using `pause()`, `resume()`, and `cancel()` respectively.

### Firmware Upgrade Mode

McuManager firmware upgrades can actually be performed in few different ways. These different upgrade 
modes determine the commands sent after the upload step. The `FirmwareUpgradeManager` can be 
configured to perform these different methods using `setMode(FirmwareUpgradeManager.Mode mode)`. 
The different firmware upgrade modes are as follows:

* **`TEST_AND_CONFIRM`**: This mode is the **default and recommended mode** for performing upgrades 
  due to it's ability to recover from a bad firmware upgrade. 
  The process for this mode is `UPLOAD`, `TEST`, `RESET`, `CONFIRM`. 
* **`CONFIRM_ONLY`**: This mode is **not recommended**. If the device fails to boot into the new 
  image, it will not be able to recover and will need to be re-flashed. 
  The process for this mode is `UPLOAD`, `CONFIRM`, `RESET`.
* **`TEST_ONLY`**: This mode is useful if you want to run tests on the new image running before 
  confirming it manually as the primary boot image. 
  The process for this mode is `UPLOAD`, `TEST`, `RESET`.

### Firmware Upgrade State

`FirmwareUpgradeManager` acts as a simple, mostly linear state machine which is determined by the `Mode`. 
As the manager moves through the firmware upgrade process, state changes are provided through the 
`FirmwareUpgradeCallback`'s `onStateChanged` method.

The `FirmwareUpgradeManager` contains an additional state, `VALIDATE`, which precedes the upload. 
The `VALIDATE` state checks the current image state of the device in an attempt to bypass certain 
states of the firmware upgrade. For example, if the image to upload is already in slot 1 on the 
device, the `State` will skip `UPLOAD` and move directly to `TEST` (or `CONFIRM` if `Mode.CONFIRM_ONLY` 
has been set). If the uploaded image is already active, and confirmed in slot 0, the upgrade will 
succeed immediately. The `VALIDATE` state makes it easy to reattempt an upgrade without needing to 
re-upload the image or manually determine where to start.

## License

This library is licensed under the Apache 2.0 license. For more info, see the `LICENSE` file.
