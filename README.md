Taxi
====

Apache Cordova based taxi meter that is deployable to both iOS and Android

<h3>Cordova version: 4.0.0</h3>

<h3>Cordova plugins:</h3>

org.apache.cordova.device 0.2.12 "Device"<br/>
org.apache.cordova.geolocation 0.3.10 "Geolocation"<br/>
org.apache.cordova.splashscreen 0.3.4 "Splashscreen"<br/>

<h3>Settings in config.xml</h3>

splashscreen : <b>screen</b> - setting splashscreen image name for Android<br/>
Fullscreen : <b>true</b> - full screen on Android<br/>
DisallowOverscroll : <b>true</b> - disallow overscroll<br/>
Orientation : <b>portrait</b> - let user use application only in portrait<br/>
StatusBarOverlaysWebView : <b>false</b><br/>
BackupWebStorage : <b>false</b> - xCode recommendation, anyway we are not storing anything in this app<br/>
ShowSplashScreenSpinner : <b>false</b> - don't want to show loading icon on splashscreen

<h3>CSS plugins used</h3>

<b>ionic.min.css</b> - layout, styling<br/>
<b>font-awesome.min.css</b> - icons

<h3>JS plugins used</h3>
<b>jquery-2.1.1.min.js</b> - library to work with JS<br/>
<b>ionic.min.js</b> - library to make project more dynamic<br/>
<b>timer.min.js</b> - for quick time manipulation

<h1>How it works?</h1>

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<meta name="viewport" content="user-scalable=no, initial-scale=1, maximum-scale=1, minimum-scale=1" />
		<link rel="stylesheet" href="plugins/ionic-v1.0.0-beta.13/css/ionic.min.css" />
		<link rel="stylesheet" href="plugins/font-awesome-4.2.0/css/font-awesome.min.css" />
		<link rel="stylesheet" href="css/index.css" />
		<title>Taxi</title>
	</head>
	<body>
		<div class="bar bar-header bar-balanced"></div>
		<div class="main-navigation"></div>
		<div class="journey-details"></div>
		<script type="text/javascript" src="cordova.js"></script>
		<script type="text/javascript" src="plugins/jquery-2.1.1.min.js"></script>
		<script type="text/javascript" src="plugins/ionic-v1.0.0-beta.13/js/ionic.min.js"></script>
		<script type="text/javascript" src="plugins/timer.min.js"></script>
		<script type="text/javascript" src="js/index.js"></script>
	</body>
</html>
```

<h1>Result of my work...</h1>

<img src="http://wapartynetworks.co.uk/taxi/2014-10-19 21.34.29.png" alt="Loading...">
<img src="http://wapartynetworks.co.uk/taxi/2014-10-19 21.35.19.png" alt="Loading...">
<img src="http://wapartynetworks.co.uk/taxi/2014-10-19 21.35.24.png" alt="Loading...">
<img src="http://wapartynetworks.co.uk/taxi/2014-10-19 21.35.27.png" alt="Loading...">