# ibeacons phonegap app

## Forked and Reworked for Cordova with instructions by Shea Silverman
#### http://blog.sheasilverman.com


Original guide and readme forked from [Evothings website](http://evothings.com/quick-guide-to-writing-mobile-ibeacon-applications-in-javascript/).


## What are iBeacons?

iBeacons are like very small lighthouses sending out signals that can be detected by a mobile application. The app can sense if a particular beacon is near or far away.  iBeacons are typically small devices powered by battery. Applications include notifications based on position/range, for example security information, commercial offerings, ads, tourist information, museum information, etc.

[iBeacon](https://developer.apple.com/ibeacon/) is Apple's beacon technology brand name and implementation. It is based on the Bluetooth Low Energy (BLE) wireless communication standard.

The BLE standard specifies an advertising mode, which is what iBeacons use. When a BLE device is in advertisement mode it repeatedly broadcasts packets over radio. The advertisement packet contains the name of the device and a scan record that can hold a limited amount of of data. Apple uses the scan record to send out a UUID that uniquely identifies the beacon.

There are several companies that make iBeacons, like [Estimote](http://estimote.com/), [Punch Through Design](http://punchthrough.com/bean/), [Kontakt](http://kontakt.io/), and [numerous additional offerings exist](http://www.alibaba.com/countrysearch/CN/ibeacon.html). The beekn website/blog presents an [iBeacon guide](http://beekn.net/guide-to-ibeacons/).



## iBeacon APIs for mobile platforms

iOS has an [iBeacon API](https://developer.apple.com/library/ios/documentation/CoreLocation/Reference/CLLocationManager_Class/CLLocationManager/CLLocationManager.html) that you must use to scan for beacon information. Notably, the CoreBuetooth API cannot be used to detect beacons, as scan records that contain iBeacon headers are blocked by Apple.

Android and other platforms that support BLE can scan for iBeacons without any restrictions.

There are several iBeacon plugins for Cordova/PhoneGap, for example the [plugin by Peter Metz](https://github.com/petermetz/cordova-plugin-ibeacon).

While Apple place restrictions of how their own APIs can be used on iOS, other platforms can implement iBeacon libraries and applications without these restrictions. In addition, Apple does not manufacture iBeacon devices, they are a third party product. The openness of the BLE standard and the diversity of iBeacon hardware devices open up for lots or innovative beacon applications.



## Implementation overview (forked and rewritten.)

The app is developed in HTML/JavaScript. For iBeacon functionality the [cordova-ibeacon plugin](https://github.com/petermetz/cordova-plugin-ibeacon) is used.


## Tracking iBeacons - ranging vs. monitoring

To track iBeacons, you specify regions for the beacons for which you want to get notifications. Here is the code that defines the regions for the pages, the id is used to identify the page associated with a beacon:

	// Regions that define which page to show for each beacon.
	app.beaconRegions =
	[
		{
			id: 'Page1',
			uuid:'E20A39F4-73F5-4BC4-A12F-17D1AD07A961',
			major: 100,
			minor: 1
		},
		{
			id: 'Page2',
			uuid:'E20A39F4-73F5-4BC4-A12F-17D1AD07A961',
			major: 100,
			minor: 2
		},
	]

Note that you need to know the UUID of the beacons you wish to track. Same UIID can be shared by multiple beacons, in which case you can use the major and minor integer numbers to uniquely identify a beacon. It is however not mandatory to specify the major/minor numbers when tracking for beacons.



## Ranging vs. monitoring

Next we will look a the code for tracking beacons. Note that two types of tracking are used for iBeacons. Monitoring, which is enabled by **startMonitoringForRegion**, tracks the entering and exiting regions. Monitoring can be run both when the app is in the foreground and in the background, may have a low update rate, and does not contain proximity information. Ranging, enabled by **startRangingBeaconsInRegion**, works only in the foreground, has a fast update rate, and has proximity information (ProximityImmediate, ProximityNear, ProximityFar). For further details regarding iBeacons and background vs foreground modes, explore [this report from Radius Networks](http://developer.radiusnetworks.com/2013/11/13/ibeacon-monitoring-in-the-background-and-foreground.html)


The example app uses ranging to determine proximity of the relaxation beacons. However, the code also enables monitoring of beacons for demonstrational purposes. The following piece of code iterates over the regions and enables monitoring and ranging for each region:

	// Start monitoring and ranging our beacons.
	for (var r in app.beaconRegions)
	{
		var region = app.beaconRegions[r]

		var beaconRegion = new cordova.plugins.locationManager.BeaconRegion(
			region.id, region.uuid, region.major, region.minor)

		// Start monitoring.
		cordova.plugins.locationManager.startMonitoringForRegion(beaconRegion)
			.fail(console.error)
			.done()

		// Start ranging.
		cordova.plugins.locationManager.startRangingBeaconsInRegion(beaconRegion)
			.fail(console.error)
			.done()
	}

## Responding to iBeacon events

To listen for beacon events, a delegate object with callback functions is used, as is shown in the following code snippet:

	// The delegate object contains iBeacon callback functions.
	var delegate = locationManager.delegate.implement(
	{
		didDetermineStateForRegion: function(pluginResult)
		{
			//console.log('didDetermineStateForRegion: ' + JSON.stringify(pluginResult))
		},

		didStartMonitoringForRegion: function(pluginResult)
		{
			//console.log('didStartMonitoringForRegion:' + JSON.stringify(pluginResult))
		},

		didRangeBeaconsInRegion: function(pluginResult)
		{
			//console.log('didRangeBeaconsInRegion: ' + JSON.stringify(pluginResult))
			app.didRangeBeaconsInRegion(pluginResult)
		}
	})


