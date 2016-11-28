# Optimizing Battery Life
[link](https://developer.android.com/training/monitoring-device-state/index.html)

> For your app to be a good citizen, it should seek to limit its impact on the battery life of its device.   
By taking steps such as batching network requests, disabling background service updates when you lose connectivity, or reducing the rate of such updates when the battery level is low, you can ensure that the impact of your app on battery life is minimized, without compromising the user experience.



<!-- TOC -->

- [Optimizing Battery Life](#optimizing-battery-life)
- [Monitoring the Battery Level and Charging State](#monitoring-the-battery-level-and-charging-state)
    - [Determine the Current Charging State](#determine-the-current-charging-state)
    - [Monitor Changes in Charging State](#monitor-changes-in-charging-state)
    - [Determine the Current Battery Level](#determine-the-current-battery-level)
    - [Monitor Significant Changes in Battery Level](#monitor-significant-changes-in-battery-level)
- [Determining and Monitoring the Docking State and Type](#determining-and-monitoring-the-docking-state-and-type)
    - [Determine the Current Docking State](#determine-the-current-docking-state)
    - [Determine the Current Dock Type](#determine-the-current-dock-type)
    - [Monitor for Changes in the Dock State or Type](#monitor-for-changes-in-the-dock-state-or-type)
- [Determining and Monitoring the Connectivity Status](#determining-and-monitoring-the-connectivity-status)
    - [Determine if You Have an Internet Connection](#determine-if-you-have-an-internet-connection)
    - [Determine the Type of your Internet Connection](#determine-the-type-of-your-internet-connection)
    - [Monitor for Changes in Connectivity](#monitor-for-changes-in-connectivity)
- [Manipulating Broadcast Receivers On Demand](#manipulating-broadcast-receivers-on-demand)
    - [Toggle and Cascade State Change Receivers to Improve Efficiency](#toggle-and-cascade-state-change-receivers-to-improve-efficiency)

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

Prefer AC > USB > discharging.

## Monitor Changes in Charging State

Charging state changes really frequently, and is uncontrollable. We might want to receive signals when the device is being charged, and this can be done by registering a `BroadcastReceiver`. This will receive signals from the `BatteryManager`.

``` xml
<receiver android:name=".PowerConnectionReceiver">
  <intent-filter>
    <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>
    <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED"/>
  </intent-filter>
</receiver>
```

and

``` java
public class PowerConnectionReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        int status = intent.getIntExtra(BatteryManager.EXTRA_STATUS, -1);
        boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||
                            status == BatteryManager.BATTERY_STATUS_FULL;

        int chargePlug = intent.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);
        boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;
        boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;
    }
}
```

## Determine the Current Battery Level

``` java
int level = batteryStatus.getIntExtra(BatteryManager.EXTRA_LEVEL, -1);
int scale = batteryStatus.getIntExtra(BatteryManager.EXTRA_SCALE, -1);

float batteryPct = level / (float)scale;
```

## Monitor Significant Changes in Battery Level

``` xml
<receiver android:name=".BatteryLevelReceiver">
  <intent-filter>
    <action android:name="android.intent.action.BATTERY_LOW"/>
    <action android:name="android.intent.action.BATTERY_OKAY"/>
  </intent-filter>
</receiver>
```

# Determining and Monitoring the Docking State and Type

## Determine the Current Docking State

``` java
IntentFilter ifilter = new IntentFilter(Intent.ACTION_DOCK_EVENT);
Intent dockStatus = context.registerReceiver(null, ifilter);

int dockState = battery.getIntExtra(EXTRA_DOCK_STATE, -1);
boolean isDocked = dockState != Intent.EXTRA_DOCK_STATE_UNDOCKED;
```

## Determine the Current Dock Type

Four types: 

1. Car
2. Desk
3. Low-End (Analog) Desk
4. High-End (Digital) Desk

``` java
boolean isCar = dockState == EXTRA_DOCK_STATE_CAR;
boolean isDesk = dockState == EXTRA_DOCK_STATE_DESK ||
                 dockState == EXTRA_DOCK_STATE_LE_DESK ||
                 dockState == EXTRA_DOCK_STATE_HE_DESK;
```

## Monitor for Changes in the Dock State or Type

Register following to get notified: `<action android:name="android.intent.action.ACTION_DOCK_EVENT"/>`

# Determining and Monitoring the Connectivity Status

## Determine if You Have an Internet Connection

``` java
ConnectivityManager cm =
        (ConnectivityManager)context.getSystemService(Context.CONNECTIVITY_SERVICE);

NetworkInfo activeNetwork = cm.getActiveNetworkInfo();
boolean isConnected = activeNetwork != null &&
                      activeNetwork.isConnectedOrConnecting();
```

## Determine the Type of your Internet Connection

``` java
boolean isWiFi = activeNetwork.getType() == ConnectivityManager.TYPE_WIFI;
```

## Monitor for Changes in Connectivity

register [`CONNECTIVITY_ACTION`](https://developer.android.com/reference/android/net/ConnectivityManager.html#CONNECTIVITY_ACTION)

# Manipulating Broadcast Receivers On Demand

> The simplest way to monitor device state changes is to create a BroadcastReceiver for each state you're monitoring and register each of them in your application manifest. Then within each of these receivers you simply reschedule your recurring alarms based on the current device state.  
> A better approach is to disable or enable the broadcast receivers at runtime. That way you can use the receivers you declared in the manifest as passive alarms that are triggered by system events only when necessary.

## Toggle and Cascade State Change Receivers to Improve Efficiency

Use `PackageManager`.

`setComponentEnabledSetting`:  
``` java
void setComponentEnabledSetting (ComponentName componentName,   
                int newState,   
                int flags)  
```
> Set the enabled setting for a package component (activity, receiver, service, provider). This setting will override any enabled state which may have been set by the component in its manifest.

| parameters |  |
-|-
| componentName | `ComponentName`: The component to enable |  
| newState | `int`: The new enabled state for the component. The legal values for this state are: `COMPONENT_ENABLED_STATE_ENABLED`, `COMPONENT_ENABLED_STATE_DISABLED` and `COMPONENT_ENABLED_STATE_DEFAULT` The last one removes the setting, thereby restoring the component's state to whatever was set in it's manifest (or enabled, by default). |
| flags | `int`: Optional behavior flags: `DONT_KILL_APP` or 0. |

``` java
ComponentName receiver = new ComponentName(context, myReceiver.class);

PackageManager pm = context.getPackageManager();

pm.setComponentEnabledSetting(receiver,
        PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
        PackageManager.DONT_KILL_APP)
``` 