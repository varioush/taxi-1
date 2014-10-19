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

Here is all static <b>HTML</b>...

<b><div class="bar bar-header bar-balanced"></div></b> - title of application page is added using jQuery <b>html()</b> function

<b><div class="main-navigation"></div></b> and <b><div class="journey-details"></div></b> are two main pages of all application. Content to these pages arriving using jQuery <b>html()</b> function

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

<h1>And here the magic begins...</h1> 

```javascript
// onSuccess Geolocation
function onSuccess(position) {
		
		// Get Lat/Lon and stingify it
		var latitude =  position.coords.latitude + "0";
		var longitude = position.coords.longitude + "0";
		
		//Limit size of values (iOS has very long numbers, Android has less, let's make values same size)
	    $(".coordinates").html(latitude.substr(0,10) + ", " + longitude.substr(0,10));
	    
		$(".history-items").append(
			'<div class="item">' +
				'<p>' +
					'<span class="current-time">' + $(".full_timestamp").html() + '</span>' +
					'<br/>' +
					latitude.substr(0,10) + ', ' + longitude.substr(0,10) +
				'</p>' +
			'</div>'
		);
		
	    /*
	    If in a future will need any more parameters, here they are...:
	    
	    'Latitude: '          + position.coords.latitude         + '<br />' +
        'Longitude: '         + position.coords.longitude        + '<br />' +
        'Altitude: '          + position.coords.altitude         + '<br />' +
        'Accuracy: '          + position.coords.accuracy         + '<br />' +
        'Altitude Accuracy: ' + position.coords.altitudeAccuracy + '<br />' +
        'Heading: '           + position.coords.heading          + '<br />' +
        'Speed: '             + position.coords.speed            + '<br />' +
        'Timestamp: '         + position.timestamp               + '<br />' +
        */
}

// onError Callback receives a PositionError object (good to have just in case)
function onError(error) {
    console.log('code: '    + error.code    + '\n' + 'message: ' + error.message + '\n');
}

$(function() {
	
	// Set header title
	$(".bar-header").html(
		'<h1 class="title">Taxi Meter</h1>'
	);
	
	// Create main menu: Start/Stop/Continue/Finish/View Journey buttons
	$(".main-navigation").html(
		'<div class="timer-clock">' +
			'<span class="journey-started">' +
			'<i class="fa fa-taxi timer_icon"></i>' +
			'<br/><br/>' +
			'<span class="taximeter">00:00</span>' +
			'<br/><br/>' +
			'</span>' +
			'<span class="journey-finished">' +
				'<i class="fa fa-flag-checkered flag_icon"></i>' +
				'<br/><br/>' +
				'<span class="flag_text">Success!</span>' +
				'<br/><br/>' +
			'</span>' +
			'<button class="button button-positive view-journey">' +
				'View Journey' +
			'</button>' +
			'<button class="button button-balanced start-timer">' +
				'Start' +
			'</button>' +
			'<button class="button button-assertive stop-timer">' +
				'Stop' +
			'</button>' +
			'<button class="button button-balanced continue-timer">' +
				'Continue' +
			'</button>' +
			'<br/>' +
			'<button class="button button-assertive finish-timer">' +
				'Finish' +
			'</button>' +
		'</div>'
	);
	
	// Timer functionality
	var TaxiMeter = new (function() {
	
	    // Taxi Meter element on the page
	    var $stopwatch;
	    
	    // Timer speed in milliseconds
	    var incrementTime = 100;
	
	    // Current timer position in milliseconds
	    var currentTime = 0;
	    
	    // $stopwatch variable linking to taximeter class
	    $(function() {
	        $stopwatch = $('.taximeter');
	    });
		
	    // Output time and increment
	    function updateTimer() {
	        var timeString = formatTime(currentTime);
	        $stopwatch.html(timeString);
	        currentTime += incrementTime;
	        var str_ct = currentTime % 60000;
	        if(str_ct == 0) {
		        navigator.geolocation.getCurrentPosition(onSuccess, onError);
	        }
	        
	        $(".tariff").html('Mon - Fri 06:00 - 20:00 (Tariff 1)');
	        
	        var str_ct2 = currentTime % 5000;
	        if(str_ct2 == 0) {
				$.getJSON("js/fares.json", function(data) {
					var items = [];
					$.each( data, function( key, val ) {
						items[key] = val;
					});
					$(".price").html("£ " + items[currentTime]);
				});
			}
	    }
		
	    // Reset timer
	    this.resetStopwatch = function() {
	        currentTime = 0;
	        TaxiMeter.Timer.stop().once();
	    };
	    
	    // Stop timer
	    this.stopStopwatch = function() {
	        TaxiMeter.Timer.toggle();
	    };
	    
	    // Start timer/journey event
		$(".start-timer").on('click', function() {
			
			TaxiMeter.Timer = $.timer(updateTimer, incrementTime, true);
			
			$(".start-timer").hide();
			$(".view-journey").hide();
			$(".journey-finished").hide();
			$(".journey-started").show();
			$(".stop-timer").show();
			$(".bar-header").removeClass("bar-balanced").addClass("bar-assertive");
			
			// Set header title
			$(".bar-header").html(
				'<h1 class="title">Journey In Progress</h1>'
			);
			
			// Set current journey parameters page
			$(".main-navigation").append(
				'<div class="list card main_navigation_details">' +
					'<a href="#" class="item item-icon-left">' +
						'<i class="icon fa fa-clock-o"></i>' +
						'<span class="clock"></span>' +
						'<span class="full_timestamp"></span>' +
					'</a>' +
					'<a href="#" class="item item-icon-left">' +
						'<i class="icon fa fa-gbp"></i>' +
						'<span class="tariff"><i class="fa fa-spinner fa-spin"></i></span>' +
					'</a>' +
					'<a href="#" class="item item-icon-left">' +
						'<i class="icon fa fa-location-arrow"></i>' +
						'<span class="coordinates"><i class="fa fa-spinner fa-spin"></i></span>' +
					'</a>' +
					'<a href="#" class="item item-icon-left">' +
						'<i class="icon fa fa-money"></i>' +
						'<span class="price"><i class="fa fa-spinner fa-spin"></i></span>' +
					'</a>' +
				'</div>'
			);
			
			// Set clock
			$(".clock").clock({"format" : "12", "calendar" : "false"});
			$(".full_timestamp").clock({"format" : "12", "calendar" : "true"});
			// Get current location
			navigator.geolocation.getCurrentPosition(onSuccess, onError);
			
			// Set journey details page
			$(".journey-details").html(
				'<div class="list card journey-history">' +
					'<div class="history-items"></div>' +
				'</div>' +
				'<button class="button back button-positive close-view-journey">' +
					'Back' +
				'</button>'
			);
			
		});
		
		// Event to stop timer
		$(".stop-timer").on('click', function() {
			
			TaxiMeter.stopStopwatch();
			
			$(".stop-timer").hide();
			$(".continue-timer").show();
			$(".finish-timer").show();
			$(".bar-header").removeClass("bar-assertive").addClass("bar-balanced");
			$(".main_navigation_details").hide();
			
			// Set header title
			$(".bar-header").html(
				'<h1 class="title">Journey Stopped</h1>'
			);
		});
		
		// Event to continue timer
		$(".continue-timer").on('click', function() {
			
			TaxiMeter.stopStopwatch();
			
			$(".continue-timer").hide();
			$(".finish-timer").hide();
			$(".stop-timer").show();
			$(".bar-header").removeClass("bar-balanced").addClass("bar-assertive");
			$(".main_navigation_details").show();
			
			// Set header title
			$(".bar-header").html(
				'<h1 class="title">Journey In Progress</h1>'
			);
		});
		
		// Event to finish timer and show 'View Journey' button
		$(".finish-timer").on('click', function() {
			
			TaxiMeter.resetStopwatch();
			
			$(".continue-timer").hide();
			$(".finish-timer").hide();
			$(".journey-started").hide();
			$(".start-timer").show();
			$(".journey-finished").show();
			$(".view-journey").show();
			$(".bar-header").removeClass("bar-balanced").addClass("bar-positive");
			$(".main_navigation_details").remove();
			// Set header title
			$(".bar-header").html(
				'<h1 class="title">Journey Finished</h1>'
			);
		});
	
	});
	
	// View journey event
	$(".view-journey").on('click', function() {
		
		// Hide main navigation
		$(".main-navigation").hide();
		
		$(".bar-header").html(
			'<h1 class="title">Journey Details</h1>'
		);
		
		// Show page
		$(".journey-details").show();
		
		//Event to close 'View Journey' page and show main navigation again
		$(".close-view-journey").on('click', function() {
			
			$(".journey-details").hide();
			$(".main-navigation").show();
			
		});
		
	});
	
});
```

<h1>Result of my work...</h1>

<img src="http://wapartynetworks.co.uk/taxi/2014-10-19 21.34.29.png" alt="Loading...">
<img src="http://wapartynetworks.co.uk/taxi/2014-10-19 21.35.19.png" alt="Loading...">
<img src="http://wapartynetworks.co.uk/taxi/2014-10-19 21.35.24.png" alt="Loading...">
<img src="http://wapartynetworks.co.uk/taxi/2014-10-19 21.35.27.png" alt="Loading...">