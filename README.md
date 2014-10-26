Taxi
====

Apache Cordova based taxi meter that is deployable to both iOS and Android

<h3>Cordova version: 4.0.0</h3>
<h3>Phones used for testing: iPhone 6 and Samsung Galaxy S4</h3>

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

Class <b>bar-header</b> - title of application page, added using jQuery <b>html()</b> function.<br/>
Class <b>main-navigation</b> and <b>journey-details</b> are two main pages of all application. Content to these pages arrives using jQuery <b>html()</b> function.

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
		
		/*
		Function to get price, required variables:
			var time 	= current time in minutes
			var tariff 	= current tariff
		*/
		
		function price(time, tariff) {
			
			// There is a minimum fare of £2.40 at all times
			var minimum = 2.40;
			
			// Tariff 1 (Monday to Friday 6am - 8pm)
			if(tariff === 1) {
				
				// Check if current journey time is less than 6 minutes
				if(time < 6) {
					// 0 - 6 minutes journey with price from £2.40 (minimum fare) to £5.60
					return calculate(minimum, time, 0, 6, 2.40, 5.60);
				}
				
				// 6 - 13 minutes journey with price from £5.60 to £8.80
				if(time >= 6 && time <= 13) {
					return calculate(minimum, time, 6, 13, 5.60, 8.80);
				}
				
				// 10 - 20 minutes journey with price from £8.60 to £13.80
				if(time >= 10 && time <= 20) {
					return calculate(minimum, time, 10, 20, 8.60, 13.80);
				}
				
				// 16 - 30 minutes journey with price from £15.00 to £22.00
				if(time >= 16 && time <= 30) {
					return calculate(minimum, time, 16, 30, 15.00, 22.00);
				}
				
				// 28 - 40 minutes journey with price from £23.00 to £29.00
				if(time >= 28 && time <= 40) {
					return calculate(minimum, time, 28, 40, 23.00, 29.00);
				}
				
				// 30 - 60 minutes journey with price from £46.00 to £85.00
				if(time >= 30 && time <= 60) {
					return calculate(minimum, time, 30, 60, 46.00, 85.00);
				}
				
			}
			
			// Tariff 2 (Monday to Friday 8pm - 10pm; Saturday and Sunday 6am - 10pm)
			if(tariff === 2) {
				
				// Check if current journey time is less than 6 minutes
				if(time < 6) {
					// 0 - 6 minutes journey with price from £2.40 (minimum fare) to £5.60
					return calculate(minimum, time, 0, 6, 2.40, 5.60);
				}
				
				// 6 - 13 minutes journey with price from £5.60 to £9.00
				if(time >= 6 && time <= 13) {
					return calculate(minimum, time, 6, 13, 5.60, 9.00);
				}
				
				// 10 - 20 minutes journey with price from £9.00 to £14.00
				if(time >= 10 && time <= 20) {
					return calculate(minimum, time, 10, 20, 9.00, 14.00);
				}
				
				// 16 - 30 minutes journey with price from £16.00 to £22.00
				if(time >= 16 && time <= 30) {
					return calculate(minimum, time, 16, 30, 16.00, 22.00);
				}
				
				// 28 - 40 minutes journey with price from £28.00 to £32.00
				if(time >= 28 && time <= 40) {
					return calculate(minimum, time, 28, 40, 28.00, 32.00);
				}
				
				// 30 - 60 minutes journey with price from £46.00 to £85.00
				if(time >= 30 && time <= 60) {
					return calculate(minimum, time, 30, 60, 46.00, 85.00);
				}
			}
			
			// Tariff 3 (Every night 10pm - 6am; Public holidays)
			if(tariff === 3) {
				
				// Check if current journey time is less than 6 minutes
				if(time < 6) {
					// 0 - 6 minutes journey with price from £2.40 (minimum fare) to £6.80
					return calculate(minimum, time, 0, 6, 2.40, 6.80);
				}
				
				// 6 - 13 minutes journey with price from £6.80 to £9.00
				if(time >= 6 && time <= 13) {
					return calculate(minimum, time, 6, 13, 6.80, 9.00);
				}
				
				// 10 - 20 minutes journey with price from £10.40 to £14.80
				if(time >= 10 && time <= 20) {
					return calculate(minimum, time, 10, 20, 10.40, 14.80);
				}
				
				// 16 - 30 minutes journey with price from £18.00 to £28.00
				if(time >= 16 && time <= 30) {
					return calculate(minimum, time, 16, 30, 18.00, 28.00);
				}
				
				// 28 - 40 minutes journey with price from £28.00 to £33.00
				if(time >= 28 && time <= 40) {
					return calculate(minimum, time, 28, 40, 28.00, 33.00);
				}
				
				// 30 - 60 minutes journey with price from £46.00 to £85.00
				if(time >= 30 && time <= 60) {
					return calculate(minimum, time, 30, 60, 46.00, 85.00);
				}
			}
		}
		
		/*
		Function to calculate price, required variables:
			var minimum 	= minimum fare at all times (£2.40 at this time)
			var time 		= current journey time
			
			var time_from 	= time interval (from)
			var time_to 	= time interval (to)
			
			var price_from 	= price interval (from)
			var price_to 	= price interval (to)
		*/
		function calculate(minimum, time, time_from, time_to, price_from, price_to) {
			
			// ...
			var mid_price = (((price_to) - (price_from)) / ((time_to - time_from) - time_from));
			
			// ...
			if(time == time_from) {
				var variable_new = ((((price_from - minimum) / time_from) * time) + minimum);
			}
			
			// Check if current journey time is less than 6 minutes
			if(time < 6) {
				
				// Check if current journey is 0 minutes, just starts. Minimum fare at all times apply
				if(time == 0) {
					var variable_new = minimum;
				}
				
				// Check if current journey time is more than 0 minutes, but less than 6 minutes
				else {
					
					// Calculate
					var variable_new = ((((price_from + (((price_to - price_from) / (time_to - time_from)) * (time - time_from) ) - minimum) / (time_from + (time - time_from)) * time)) + minimum);
				}
			}
			
			// Check if current journey time is more than 6 minutes
			else {
				
				// Check if current journey time is more than minumum value of time interval
				if(time > time_from) {
					
					// Calculate
					var variable_new = ((((price_from + (((price_to - price_from) / (time_to - time_from)) * (time - time_from) ) - minimum) / (time_from + (time - time_from)) * time)) + minimum);
				}
			}
			
			// Round price to 2 decimal places
			variable_new = parseFloat(Math.round(variable_new * 100) / 100).toFixed(2);
			
			/*
				If instead of 'return' would use 'append', there would be more values returned.
				It would happen because some of time intervals are merging (it still would calculate price correctly).
				
				It's possible to add some more additional calculations if we would have current distance
				
				i.e. 
				6 - 13 minutes (1 mile) with tariff £5.60 - £8.80
				10 - 20 minutes (2 miles) with tariff £8.60 - £13.80
				
				10 - 13 minutes is merged, but if we would know, that it's 2 miles journey, we would use £8.60 - £13.80 tariff for this merged time interval.
				In this case we use £5.60 - £8.80, because we are not checking distance.
				There are big jumps of price in some cases because of that, but it's still fits 'Taxi fares from 5 April 2014'
			*/
			
			// Return current price
			return variable_new;
		}
		
	    // Output time and increment
	    function updateTimer() {
	        var timeString = formatTime(currentTime);
	        $stopwatch.html(timeString);
	        currentTime += incrementTime;
	        var str_ct = currentTime % 60000;
	        if(str_ct == 0) {
		        navigator.geolocation.getCurrentPosition(onSuccess, onError);
		        
				// Date object
				var d = new Date();
				
				// Returns the year (four digits)
				var getFullYear = d.getFullYear();
				
				// Returns the month (from 0-11)
				var getMonth = d.getMonth();
				
				// Returns the day of the week (from 0-6)
				var getDay = d.getDay();
				
				// Returns the hour (from 0-23)
				var getHours = d.getHours();
				
				// Tariff 1 (Monday to Friday 6am - 8pm)
				if(getDay >= 1 && getDay <= 5 && getHours >= 6 && getHours < 20) {
					var tariff = 1;
				}
				// Tariff 2 (Monday to Friday 8pm - 10pm)
				if(getDay >= 1 && getDay <= 5 && getHours >= 20 && getHours < 22) {
					var tariff = 2;
				}
				// Tariff 2 (Saturday and Sunday 6am - 10pm)
				if(getDay == 6 || getDay == 0 && getHours >= 6 && getHours < 22) {
					var tariff = 2;
				}
				// Tariff 3 (Every night 10pm - 6am)
				if(getHours >= 22 && getHours <= 23 || getHours >= 0 && getHours < 6) {
					var tariff = 3;
				}
				// Tariff 3 (Public holidays)
				
				/*
				https://www.gov.uk/bank-holidays
				
				2014
				25 December		Thursday	Christmas Day
				26 December		Friday		Boxing Day
				
				2015
				1 January		Thursday	New Year’s Day
				3 April			Friday		Good Friday
				6 April			Monday		Easter Monday
				4 May			Monday		Early May bank holiday
				25 May			Monday		Spring bank holiday
				31 August		Monday		Summer bank holiday
				25 December		Friday		Christmas Day
				28 December		Monday		Boxing Day (substitute day)
				
				2016
				1 January		Friday		New Year’s Day
				25 March		Friday		Good Friday
				28 March		Monday		Easter Monday
				2 May			Monday		Early May bank holiday
				30 May			Monday		Spring bank holiday
				29 August		Monday		Summer bank holiday
				26 December		Monday		Boxing Day
				27 December		Tuesday		Christmas Day (substitute day)
				*/
				
				// Christmas Day
				if(getFullYear == 2014 && getMonth == 11 && getDay == 25) {
					var tariff = 3;
				}
				// Boxing Day
				if(getFullYear == 2014 && getMonth == 11 && getDay == 26) {
					var tariff = 3;
				}
				// New Year’s Day
				if(getFullYear == 2015 && getMonth == 0 && getDay == 1) {
					var tariff = 3;
				}
				// Good Friday
				if(getFullYear == 2015 && getMonth == 3 && getDay == 3) {
					var tariff = 3;
				}
				// Easter Monday
				if(getFullYear == 2015 && getMonth == 3 && getDay == 6) {
					var tariff = 3;
				}
				// Early May bank holiday
				if(getFullYear == 2015 && getMonth == 4 && getDay == 4) {
					var tariff = 3;
				}
				// Spring bank holiday
				if(getFullYear == 2015 && getMonth == 4 && getDay == 25) {
					var tariff = 3;
				}
				// Summer bank holiday
				if(getFullYear == 2015 && getMonth == 6 && getDay == 31) {
					var tariff = 3;
				}
				// Christmas Day
				if(getFullYear == 2015 && getMonth == 11 && getDay == 25) {
					var tariff = 3;
				}
				// Boxing Day (substitute day)
				if(getFullYear == 2015 && getMonth == 11 && getDay == 28) {
					var tariff = 3;
				}
				// New Year’s Day
				if(getFullYear == 2016 && getMonth == 0 && getDay == 1) {
					var tariff = 3;
				}
				// Good Friday
				if(getFullYear == 2016 && getMonth == 2 && getDay == 25) {
					var tariff = 3;
				}
				// Easter Monday
				if(getFullYear == 2016 && getMonth == 2 && getDay == 28) {
					var tariff = 3;
				}
				// Early May bank holiday
				if(getFullYear == 2016 && getMonth == 4 && getDay == 2) {
					var tariff = 3;
				}
				// Spring bank holiday
				if(getFullYear == 2016 && getMonth == 4 && getDay == 30) {
					var tariff = 3;
				}
				// Summer bank holiday
				if(getFullYear == 2016 && getMonth == 6 && getDay == 29) {
					var tariff = 3;
				}
				// Boxing Day
				if(getFullYear == 2016 && getMonth == 11 && getDay == 26) {
					var tariff = 3;
				}
				// Christmas Day (substitute day)
				if(getFullYear == 2016 && getMonth == 11 && getDay == 28) {
					var tariff = 3;
				}
				
				// Update tariff
				if(tariff === 1) {
					$(".tariff").html('Tariff 1');
				}
				if(tariff === 2) {
					$(".tariff").html('Tariff 2');
				}
				if(tariff === 3) {
					$(".tariff").html('Tariff 3');
				}
				
				var currentTimeMinutes = Math.round(currentTime / 60000);
				$(".price").html("£ " + price(currentTimeMinutes, tariff));
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
			
			// Date object
			var d = new Date();
			
			// Returns the year (four digits)
			var getFullYear = d.getFullYear();
			
			// Returns the month (from 0-11)
			var getMonth = d.getMonth();
			
			// Returns the day of the week (from 0-6)
			var getDay = d.getDay();
			
			// Returns the hour (from 0-23)
			var getHours = d.getHours();
			
			// Tariff 1 (Monday to Friday 6am - 8pm)
			if(getDay >= 1 && getDay <= 5 && getHours >= 6 && getHours < 20) {
				var tariff = 1;
			}
			// Tariff 2 (Monday to Friday 8pm - 10pm)
			if(getDay >= 1 && getDay <= 5 && getHours >= 20 && getHours < 22) {
				var tariff = 2;
			}
			// Tariff 2 (Saturday and Sunday 6am - 10pm)
			if(getDay == 6 || getDay == 0 && getHours >= 6 && getHours < 22) {
				var tariff = 2;
			}
			// Tariff 3 (Every night 10pm - 6am)
			if(getHours >= 22 && getHours <= 23 || getHours >= 0 && getHours < 6) {
				var tariff = 3;
			}
			// Tariff 3 (Public holidays)
			
			/*
			https://www.gov.uk/bank-holidays
			
			2014
			25 December		Thursday	Christmas Day
			26 December		Friday		Boxing Day
			
			2015
			1 January		Thursday	New Year’s Day
			3 April			Friday		Good Friday
			6 April			Monday		Easter Monday
			4 May			Monday		Early May bank holiday
			25 May			Monday		Spring bank holiday
			31 August		Monday		Summer bank holiday
			25 December		Friday		Christmas Day
			28 December		Monday		Boxing Day (substitute day)
			
			2016
			1 January		Friday		New Year’s Day
			25 March		Friday		Good Friday
			28 March		Monday		Easter Monday
			2 May			Monday		Early May bank holiday
			30 May			Monday		Spring bank holiday
			29 August		Monday		Summer bank holiday
			26 December		Monday		Boxing Day
			27 December		Tuesday		Christmas Day (substitute day)
			*/
			
			// Christmas Day
			if(getFullYear == 2014 && getMonth == 11 && getDay == 25) {
				var tariff = 3;
			}
			// Boxing Day
			if(getFullYear == 2014 && getMonth == 11 && getDay == 26) {
				var tariff = 3;
			}
			// New Year’s Day
			if(getFullYear == 2015 && getMonth == 0 && getDay == 1) {
				var tariff = 3;
			}
			// Good Friday
			if(getFullYear == 2015 && getMonth == 3 && getDay == 3) {
				var tariff = 3;
			}
			// Easter Monday
			if(getFullYear == 2015 && getMonth == 3 && getDay == 6) {
				var tariff = 3;
			}
			// Early May bank holiday
			if(getFullYear == 2015 && getMonth == 4 && getDay == 4) {
				var tariff = 3;
			}
			// Spring bank holiday
			if(getFullYear == 2015 && getMonth == 4 && getDay == 25) {
				var tariff = 3;
			}
			// Summer bank holiday
			if(getFullYear == 2015 && getMonth == 6 && getDay == 31) {
				var tariff = 3;
			}
			// Christmas Day
			if(getFullYear == 2015 && getMonth == 11 && getDay == 25) {
				var tariff = 3;
			}
			// Boxing Day (substitute day)
			if(getFullYear == 2015 && getMonth == 11 && getDay == 28) {
				var tariff = 3;
			}
			// New Year’s Day
			if(getFullYear == 2016 && getMonth == 0 && getDay == 1) {
				var tariff = 3;
			}
			// Good Friday
			if(getFullYear == 2016 && getMonth == 2 && getDay == 25) {
				var tariff = 3;
			}
			// Easter Monday
			if(getFullYear == 2016 && getMonth == 2 && getDay == 28) {
				var tariff = 3;
			}
			// Early May bank holiday
			if(getFullYear == 2016 && getMonth == 4 && getDay == 2) {
				var tariff = 3;
			}
			// Spring bank holiday
			if(getFullYear == 2016 && getMonth == 4 && getDay == 30) {
				var tariff = 3;
			}
			// Summer bank holiday
			if(getFullYear == 2016 && getMonth == 6 && getDay == 29) {
				var tariff = 3;
			}
			// Boxing Day
			if(getFullYear == 2016 && getMonth == 11 && getDay == 26) {
				var tariff = 3;
			}
			// Christmas Day (substitute day)
			if(getFullYear == 2016 && getMonth == 11 && getDay == 28) {
				var tariff = 3;
			}
			
			// Update tariff
			if(tariff === 1) {
				$(".tariff").html('Tariff 1');
			}
			if(tariff === 2) {
				$(".tariff").html('Tariff 2');
			}
			if(tariff === 3) {
				$(".tariff").html('Tariff 3');
			}
			
			var currentTimeMinutes = Math.round(currentTime / 60000);
			$(".price").html("£ " + price(currentTimeMinutes, tariff));
			
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