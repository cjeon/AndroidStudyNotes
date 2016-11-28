# Optimizing Battery Life
[link](https://developer.android.com/training/monitoring-device-state/index.html)

> For your app to be a good citizen, it should seek to limit its impact on the battery life of its device.   
By taking steps such as batching network requests, disabling background service updates when you lose connectivity, or reducing the rate of such updates when the battery level is low, you can ensure that the impact of your app on battery life is minimized, without compromising the user experience.



<!-- TOC -->

- [Optimizing Battery Life](#optimizing-battery-life)
- [Monitoring the Battery Level and Charging State](#monitoring-the-battery-level-and-charging-state)
    - [Determine the Current Charging State](#determine-the-current-charging-state)

<!-- /TOC -->

# Monitoring the Battery Level and Charging State

You can minimize the battery impact by performing heavy operations only when the device is charging. 

## Determine the Current Charging State

Create an `IntentFilter` with `Intent.ANCTION_BATTERY_CHARGED` and register it to context.

``` java
// Create IntentFilter
IntentFilter ifilter = new IntentFilter(Intent.ACTION_BATTERY_CHANGED);
Intent batteryStatus = context.registerReceiver(null, ifilter);

// get isCharging
int status = batteryStatus.getIntExtra(BatteryManager.EXTRA_STATUS, -1);
boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||
                     status == BatteryManager.BATTERY_STATUS_FULL;

// get USB/AC
int chargePlug = batteryStatus.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);
boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;
boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;
```


                     