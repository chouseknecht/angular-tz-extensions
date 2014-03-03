# Angular TZ Extensions

Javascript is good at creating a date with the local timezone, providing a robust set of methods for returning date and time information for the current timezone and UTC. But what if you want the date aligned to a timezone other than the local timezone? What if you need a list of timezones for a user to choose from? What if you need to know the offset for a particular timezone? Angular TZ Extensions provides the solution.

Originally forked from  https://github.com/michaelahlers/angular-timezones.

## Install 

Install using [Bower](https://github.com/bower/bower):

    bower install angular-tz-extensions

Once installed include the follwing scripts in your app:

    <script type="text/javascript" src="/bower_components/timezone-js/src/date.js"></script>
    <script type="text/javascript" src="/bower_components/angular-tz-extensions/lib/angular-tz-extensions.js"></script>

If you want to detect the local timezone, include the following as well:

	<script type="text/javascript" src="/packages/jstimezonedetect/jstz.min.js"></script>
    
## Usage

After including `angular-timezones.js`, add this package to your application.

    var yourApplication = angular.module('your-application', ['Timezones'])

### Configuration

This package provides the [IANA timezone data](http://iana.org/time-zones), but you may have this resource served from a different location or you may wish to provide your own data. To change that location, set the `$timezones.definitions.location` property to the appropriate path or URL.

    yourApplication.constant('$timezones.definitions.location', '/custom/path/to/tz/data')

This is done by the unit tests and illustrated in the included sample app (see Examples below).

### Align date to a given timezone

Use `$timezones.align(timezone, date)` to align a date object to a timezone represeneted as an Olsan timezone string value. The getFullYear, getMonth, getDate, getHours, getMinutes, getSeconds and getTimezone methods of the returned date object will present values in the requested timezone.

Below is an example comparing a date object aligned to the local timezone (America/New_York) with a date object created using the align method a timezone of 'America/Los_Angeles':

	var rightNow = new Date();
	console.log(rightNow.getTimezoneOffset());
	console.log(rightNow.getHours());
	console.log($filter('date')(rightNow,"yyyy-MM-dd HH:mm:ss Z"));

	var test = $timezones.align(rightNow, 'America/Los_Angeles'); 
	console.log(test.timezone);
	console.log(test.getTimezoneOffset());
	console.log(test.getHours());
	console.log($filter('date')(test,"yyyy-MM-dd HH:mm:ss Z"));

Results in:

	300 
	14
	2014-03-03 14:40:33 -0500
	
	America/Los_Angeles
	480
	11
	2014-03-03 11:40:33 -0800

Note that TimezoneJS (timezone-js/src/date.js) adds additional properties and methods to the date object. Here we're accessing the timezone property. There is also a getTimezoneInfo() method. See TimezoneJS documentation for more details. 

Alignment can also be accomplished at the view level using the provided tzAlign filter:

			{{ someDate|tzAlign:'America/Los_Angeles'|date:"yyyy-MM-dd HH:mm:ss Z" }}

### Resolution

The `$timezones.resolve(timezone, reference)` function will, using the reference `Date` provided, look up complete details about the timezone&mdash;including the abbreviation, offset, and decomposed region and locality. This is useful for avoiding clever tricks to extract abbreviations from `Date.toString` (which is not particularly portable or robust). Additionally, resulting values are derived from the authoritative IANA timezone data.
	
    var scope.timezone = $timezone.resolve('America/New_York', new Date());
    console.log(scope.timezone);

Returns:
    
    {
        abbreviation: 'EST',
        locality: 'New York',
        name: 'America/New_York',
        offset: 300,
        region: 'America'
    }

### Detection

If [jsTimezoneDetect](https://bitbucket.org/pellepim/jstimezonedetect) is included, the `$timezones.getLocal()` function will detect the browser's local timezone and provide a complete definition that's resolved against the IANA database. For convenience, jsTimezoneDetect is included in packages/jstimezonedetect. You may want to pull the latest version in from bitbucket.



### List of available timezones

You can retrieve an array of all available timezones- perfect for populating a select element. Use the `$timezone.getZoneList($scope)` method, passing in a scope instance. The method reads the zone.tab tab file, which is part of tzdata. When the data is ready, the method emits 'zonesReady'. Retrieve the data inside `$scope.$on('zonesReady', callback)`. The data will be available in local storage and can be accessed using: `JSON.parse(localStorage.zones)`. Here's an example taken from the included sample app:  
    
    if ($scope.removeZonesReady) {
        $scope.removeZonesReady();
    }
    $scope.removeZonesReady = $scope.$on('zonesReady', function() {
        var i;
        $scope.zones = JSON.parse(localStorage.zones);
        $scope.current_timezone = $timezones.getLocal();
        for (i=0; i < $scope.zones.length; i++) {
            if ($scope.zones[i].name === $scope.current_timezone.name) {
                $scope.selectedZone = $scope.zones[i];
                break;
            }
        }
    });

    $timezones.getZoneList($scope);

## Examples

A sample application is included. Run it locally using [Node](http://nodejs.org):

    node ./scripts/web-server.js

Once running, point your browser to http://localhost:8000/app/filters.html

## Developers

_Timezones for Angular_ is tested with [Karma](http://karma-runner.github.io/) and [PhantomJS](http://phantomjs.org/) against the latest available release of [jQuery](http://jquery.com/) (2.0.0) and [AngularJS](http://angularjs.com/) (1.1.4).

With [NPM](http://npmjs.com/) installed, you can test your modifications with the following.

    npm install
    npm test
