# Change Log
## [Unreleased]
- [Fixed] `http` event `success` callback receives `{}` but `error` receives `String` (issue #36).
- [Changed] Add ability to set odometer to any arbitrary value.  Before, odometer could only be reset to `0` via `resetOdometer`.  The plugin now uses `setOdometer(Float, successFn, failureFn`.  `resetOdometer` is now just an alias for `setOdometer(0)`.  `setOdometer` will now internally perform a `#getCurrentPosition`, so it can know the exact location where the odometer was set at.  As a result, using `#setOdometer` is exactly like performing a `#getCurrentPosition` and the `success` / `failure` callbacks use the same method-signature, where the `success` callback is provided the `location`.
- [Added] Added ability to create your own arbitrary **background tasks** with new `#startBackgroundTask` method.  Some of the plugin's methods receive a `taskId` which you've had to call `bgGeo.finish(taskId)` upon.  These automatically created `taskId` will soon be removed.  It will be **up to you** to create your own as desired, when you need to perform any long-running task in any of the plugin's callbacks.  `#finish` operates in the same manner as before.

## [1.3.0] - 2016-11-23
- [Fixed] `stopWatchPostion` callbacks not being called.
- [Fixed] Bug with Android geofences not posting `event: geofence` and the actual `geofence` data was missing (The data sent to Javascript callback was ok, just the data sent to HTTP.
- [Fixed] Logic bug in `TSGeofenceManager`; was not performing geospatial query when changing state from **MOVING -> STATIONARY**.
- [Added] Geofences-only mode for both iOS and Android **BETA**.  Start geofences-only mode with method `#startGeofences`.
- [Changed] Add some intelligence to iOS motion-detection system:  Use a Timer of `activityRecognitionInterval` seconds before engaging location-services after motion is detected.  This helps to reduce false-positives, particularly when using `preventSuspend` while walking around one's house or office.
- [Changed] Add more intelligence to iOS motion-detection system:  The plugin will be **eager** to engage the stop-detection, as soon as it detects `still`, regardless of confidence.  When the plugin is currently in the **moving** state and detects `still`, it will engage a timer of `activityRecognitionInterval` milliseconds -- when this timer expires and the motion-detector still reports `still`, the stop-detection system will be engaged.  If any *moving* type activity occurs during this time, the timer will be cancelled.
- [Fixed] Bug in Android Scheduler, failing to `startOnBoot`.  Issue #985
- [Added] `#removeListeners` method.  Removes all listeners registered with plugin via `#on` method.
- [Changed] With `preventSuspend: true`, the plugin will no longer immediately engage location-services as soon as it sees a "moving"-type motion-activity:  it will now calculate if the current position is beyond stationary geofence. This helps reduce false-positives engaging location-services while simply walking around one's home or office.
- [Fixed] iOS `batchSync`: When only 1 record in batch, iOS fails to pack the records in a JSON `location: []`, appending to a `location: {}` instead.
- [Fixed] Android was only handling the first geofence event when multiple geofences fire simultaneously.
- [Changed] The plugin will ignore `autoSyncThreshold` when a `motionchange` event occurs.
- [Fixed] Fixed ui-blocking issue when plugin boots with locations in its database with `autoSync: true`.  Found a case where the plugin was executing HTTP Service on the UI thread.
- [Fixed] Return current `state {Object}` in callback to `setConfig`.
- [Fixed] iOS Scheduler puked when provided with a `null` or `[]` schedule.
- [Changed] iOS Scheduler behaviour changed to match Android, where `#stopSchedule` does **not** execute `#stop` on the plugin itself.
- [Fixed] FMDB [has issues](https://github.com/ccgus/fmdb/pull/180) binding array arguments (eg: DELETE FROM locations WHERE id IN(?)).  Solution is to simply compose the query string with concatenation.  Sanitization isn't required here anyway, since the ids come directly from my own query.

## [1.2.5] - 2016-10-31
- [Fixed] Issue compiling demo with `typescript@2.x`
- [Fixed] Remove Android deprecation issues with `AndroidManifest.xml`
- [Changed] Refactor Android scheduler to no use a Service.

## [1.2.4] - 2016-10-27
- [Fixed] Bug in Android `#insertLocation` (Reported in issue #23)
- [Fixed] iOS geofencing issue where multiple geofences trigger simultaneously, only the last geofence event would be transmitted to the client and persisted to database.
- [Added] iOS can now initial-trigger a geofence when device is already within the newly added geofence.  Android has always had this behavour by default.  This behaviour is controlled with new `@config {Integer meters} geofenceInitialTriggerEntry [true]`.
- [Fixed] Android had a bug when Activity terminated with `stopOnTerminate: true`

## [1.2.3] - 2016-10-20
- [Changed] Implement database-logging for Android.  Both iOS and Android now send logs to the database.

## [1.2.2] - 2016-10-16
- [Changed] Remove `nativescript-background-fetch` from dependencies.  Users will have to manually `tns plugin add nativescript-background-fetch`, since `tns plugin add nativescript-background-geolocation-lt` doesn't add `background-fetch` the the root `node_modules` folder.  This fixes the problem referencing background-fetch's .podspec file.

## [1.2.1] - 2016-10-14
- [Changed] Refactor typescript API.  `BackgroundGeolocation` API is all static methods now -- You no longer create an instance of `BackgroundGeolocation`
**OLD**
```Javascript
var bgGeo = new BackgroundGeolocation();
bgGeo.configure(config, callback);

```
**NEW**
```Javascript
BackgroundGeolocation.configure(config, callback);
```

This will make interacting with the plugin throughout your views **much** easier, since views are destroyed when navigated away from.

- [Fixed] `package.json` now references a definitions file.  This should solve issue with error reported in issue #18:
```
JS ERROR Error: Could not find module 'nativescript-background-geolocation-lt'
```

## [1.2.0] - 2016-10-11
- [Changed] Refactor iOS Logging system to use popular CocoaLumberjack library.  iOS logs are now stored in the database!  By default, logs are stored for 3 days, but is configurable with `logMaxDays`.  Logs can now be filtered by logLevel:

| logLevel | Label |
|---|---|
|`0`|`LOG_LEVEL_OFF`|
|`1`|`LOG_LEVEL_ERROR`|
|`2`|`LOG_LEVEL_WARNING`|
|`3`|`LOG_LEVEL_INFO`|
|`4`|`LOG_LEVEL_DEBUG`|
|`5`|`LOG_LEVEL_VERBOSE`|

`#getLog`, `#emailLog` operate in the same manner as before.

- [Fixed] If user declines "Motion Activity" permission, plugin failed to detect this authorization failure and fallback to the accelerometer-based motion-detection system.

- [Changed] Refactored Geolocation system.  The plugin is no longer bound by native platform limits on number of geofences which can be monitored (iOS: 20; Android: 100).  You may now monitor infinite geofences.  The plugin now stores geofences in its SQLite db and performs a geospatial query, activating only those geofences in proximity of the device (@config #geofenceProximityRadius, @event `geofenceschange`).  See the new [Geofencing Guide](./docs/geofencing.md)

## [1.1.1] - 2016-09-25
- [Fixed] Bugs in preventSuspend during background-fetch event

## [1.1.0]
- [Changed] Upgrade to nativescript-2.3.0
- [Fixed] Bug in prevent-suspend where the plugin failed to re-start its prevent-suspend timer if no MotionActivity event occurred during that interval.  Prevent-suspend system should now operate completely independently of MotionDetector.
- [Fixed] `#stop` method wasn't calling `stopMonitoringSignificantChanges`, resulting in location-services icon failing to toggle OFF.
- [Fixed] Issue where iOS crashes when configured with null url.
- [Added] iOS `watchPosition` mechanism.
- [Changed] Refactored iOS motion-detection system.  Improved iOS motion-triggering when using `CMMotionActivityManager` (ie: when not using `disableMotionActivityUpdates: true`).  iOS can now trigger out of stationary-mode just like android, where it sees a 'moving-type' motion-activity (eg: 'on_foot', 'in_vehicle', etc).  Note: this will still occur only when your app isn't suspended (eg: app is in foreground, `preventSuspend: true`, or `#watchPosition` is engaged).
- [Changed] Refactored iOS "prevent suspend" system to be more robust.
- [Fixed] iOS locations sent to Javascript client had a different `uuid` than the one persisted to database (and synced to server).
-[Added] new iOS 10 .plist required key for accelerometer updates `NSMmotionUsageDescription` to `config.xml`.
- [Added] New required android permission `<uses-feature android:name="android.hardware.location.gps" />`.
- [Fixed] `removeGeofences` was removing stationary-region.  This would prevent stationary-exit if called while device is in stationary-mode
- [Fixed] Android pukes when it receives an empty schedule `[]`.
- [Fixed] Android when configured with `batchSync: true, autoSync: true` was failing because the plugin automatically tweaked `autoSync: false` but failed to reset it to the configured value.  This behaviour was obsolete and has been removed.
- [Added] Add new config `@param {Integer} autoSyncThreshold [0]`.  Allows you to specify a minimum number of persisted records to trigger an auto-sync action.
- [Fixed] Issue #837.  Android `SimpleDateFormat` used for rendering location timestamp was not being used in a thread-safe manner, resulting in corrupted timestamps for some
- [Fixed] Issue #804, null pointer exeception on mGoogleApiClient
- [Fixed] Issue #806.  PlayServices connect error event was fired before listeners arrive; Dialog to fix problem was never shown.
- [Changed] Removed `app-compat` from Gradle dependencies.
- [Changed] Fire http error callback when HTTP request is not 200ish (ie: 200, 201, 204).  Fixes issue #819.  Contradicts #774.
- [Changed] Remove `play-services:app-compat-v7` from Gradle dependencies
- [Fixed] Android heartbeat location wasn't having its meta-data updated (ie: `event: 'heartbeat', battery:<current-data>, uuid: <new uuid>`)
- [Changed] Reduce Android `minimumActivityRecognitionConfidence` default from `80` to `75` (issue #825)
- [Changed] Android will ask for location-permission when `#configure` is executed, rather than waiting for `#start`.
- [Changeed] Android will catch `java.lang.SecurityException` when attempting to request location-updates without "Location Permission"

## [1.0.4] - 2016-08-01
- Fix bug in Demo

## [1.0.3] - 2016-08-01
- Android & iOS are both working well and nearly ready for action.

## [1.0.2]

## [1.0.1]

## [1.0.0]

