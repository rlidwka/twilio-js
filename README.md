twilio-js
=========

The Twilio API and TwiML for node.js

## Installation

The library will be availble on npm when released

<pre>npm install slow_down_fast_lane_its_not_ready_yet</pre>

Please use the Github issue tracker to report any issues or bugs you uncover.

## Usage

Require the library in your node application as

```javascript
var Twilio = require('twilio-js');
```

## Configuration

Before invoking any functions that interace with the API, you must set your account SID and auth token. This is done with properties on the `Twilio` object

```javascript
Twilio.AccountSid = "ACxxxxxxxxxxxxxxxxxxxxxxx";
Twilio.AuthToken  = "xxxxxxxxxxxxxxxxxxxxxxxxx";
```

TODO: Raise specific error if API related functions called before account credentials configure?

# Getting started

## Summary

Twilio resources are represented as JavaScript object, e.g. `Twilio.SMS` and operations on those resources are performed via functions that are properties of those objects, e.g. `Twilio.SMS.create`

Resources that can be created via the API, using the HTTP POST verb can be done so in the library using the `.create` function, e.g.

```javascript
Twilio.Call.create({to: "+12125551234", from: "+16465551234", url: "http://example.com/voice"}, function(err,res) {
  // Your code
})
```
When a response is received from the API, the supplied callback function is invoked with an object representation of the resource passed in as the `res` argument as illustrated here. In the case of an error level response, an `Error` object will passed in as the `err` argument.

Resources that can be removed via the API, using the HTTP DELETE verb can be done so in the library using the `destroy` function on the resource object representation, e.g.

```javascript
// Delete all log entries
Twilio.Notification.all(function(err, res) {
  res.forEach(function(obj,i,arr) { obj.destroy() }
}

The object representations yielded to the callback functions have properties that correspond to those of the resource. The Twilio API documentation itself is the canonical reference for which resources have what properties, and which of those can be updated by the API. Please refer to the Twilio REST API documentation for thos information.

## Accessing resource instances

Resource instances can be accessed ad hoc passsing the resource sid to the `.find` class method on the resource class, e.g.

```javascript
Twilio.Call.find('CAe1644a7eed5088b159577c5802d8be38', function(err, res) {})
```

This will yield an object with propertes corresponding to the attributes of the resource. The properties are camel-cased.

## Querying list resources

List resources can be accessed ad hoc by calling the `.all` class method on the resource class, e.g.

```javascript
Twilio.Call.all(function(err, res) {})
```

This will return a collection of objects, each a representation of the corresponding resource.

### Using filter parameters to refine a query

The `.all` function will ask Twilio for all resource instances on that list resource, this can easily result in a useless response if there are numerous resource instances on a given resource. The `.all` class method accepts an optional object of options for parameters to filter the response, e.g.

```javascript
Twilio.Call.all({ from: "+12125550000" }, function(err, res) {})
```

Twilio does some fancy stuff to implement date ranges, consider the API request:

<pre>GET /2010-04-01/Accounts/AC5ef87.../Calls?StartTime&gt;=2009-07-06&EndTime&lt;=2009-07-10</pre>

This will return all calls started after midnight January 01st 2012 and completed before July 10th 2009. To make the same reqest using this library:

```javascript
Twilio.Call.all({ startedBefore: "2012-12-01" }, function(err, res) {})
```

### Pagination

The Twilio API paginates API responses and by default it will return 30 objects in one response, this can be overridden to return up to a maximum of 1000 per response using the `:page_size` option, If more than 1000 resources instances exist, the `:page` option is available, e.g.

```javascript
Twilio.Call.all({ pageSize: 1000, page: 7 }, function(err, res) {})
```
## Updating resource attributes

Certain resources have attributes that can be updated with the REST API. Instances of those resources can be updated by changing the properties on the response object and calling the save function.

```javascript
Twilio.Call.all({ status: 'in-progress' }, function(err, res) {
  var call = res[0]
  call.url = 'http://example.com/in_ur_apiz_hijackin_ur_callz.xml'
  call.save() // TODO: is there a case for this to accept a callback function?
})
```
# Twilio Client

To generate capability tokens for use with Twilio Client you can use `Twilio::CapabilityToken.create`

```javascript
Twilio.CapabilityToken.create({
  allow_incoming: 'unique_identifier_for_this_user',
  allow_outgoing: 'your_application_sid'
})
```

You can create capability tokens on arbitrary accounts, e.g. subaccounts. Just pass in those details:

```javascript
Twilio.CapabilityToken.create({
  account_sid:    'AC00000000000000000000000',
  auth_token:     'XXXXXXXXXXXXXXXXXXXXXXXXX',
  allow_incoming: 'unique_identifier_for_this_user',
  allow_outgoing: 'your_application_sid'
})
```

You can also pass arbitrary parameters into your outgoing privilege, these are sent from Twilio as HTTP request params when it hits your app endpoint for TwiML.

```javascript
Twilio.CapabilityToken.create({allow_outgoing: ['your_application_sid', { foo: 'bar' }]}]
```

By default tokens expire exactly one hour from the time they are generated. You can choose your own token ttl like so:

```javascript
Twilio.CapabilityToken.create({
  allow_incoming: 'unique_identifier_for_this_user',
  allow_outgoing: 'your_application_sid',
  expires:        // TODO: how to denote time. ISO or epoch?
})
```

# Twilio Connect

With Twilio Connect you can attribute Twilio usage to accounts of customers that have authorised you to perform API calls on there behalf. twilio-rb supports Twilio Connect. To make an API call using a Twilio Connect account, two extra parameters are required, `account_sid` and `connect`

```javascript
Twilio::SMS.create({
  to: '+12125551234', from: '+6165550000',
  body: 'this will not be billed to the application developer',
  account_sid: CONNECT_ACCOUNT_SID, connect: true
})
```

# Subaccounts

The Twilio REST API supports subaccounts that is discrete accounts owned by a master account. twilio-js supports this too.

## Subaccount creation

You can create new subaccounts by using `Twilio.Account.create()`

## Performing actions on resources belonging to subaccounts

There are three ways to perform an operation on an account other than the master account: you can pass in the subaccount sid

```javascript
Twilio::SMS.create({
  to: '+12125551234', from: '+6165550000',
  body:  'This will be billed to a subaccount, sucka!',
  account_sid: 'ACXXXXXXXXXXXXXXXXXXXXXXXX'
})
```

# Building TwiML documents

A TwiML document is an XML document.

The following js code:

```javascript
Twilio.TwiML.build(function(res) {
    res.say('Hey man! Listen to this!', { voice: 'man' });
    res.play('http://foo.com/cowbell.mp3');
    res.say('What did you think of that?', { voice: 'man' });

    res.record({
      action: "http://foo.com/handleRecording.php",
      method: "GET", maxLength: "20", finishOnKey: "*"
    })

    res.gather(function(res) { res.say('Now hit some buttons!') }, { action: "/process_gather.php", method: "GET" })
    res.say('Awesome! Thanks!', { voice: 'man', language: 'en-gb' });
    res.hangup()
})
```

Therefore emits the following TwiML document:

<pre><?xml version="1.0" encoding="UTF-8"?>
&lt;Response&gt;
  &lt;Say voice="man"&gt;Hey man! Listen to this!&lt;/Say&gt;
  &lt;Play&gt;http://foo.com/cowbell.mp3&lt;/Play&gt;
  &lt;Say voice="man"&gt;What did you think of that?!&lt;/Say&gt;
  &lt;Record maxLength="20" method="GET" action="http://foo.com/handleRecording.php" finishOnKey="*"/&gt;
  &lt;Gather method="GET" action="/process_gather.php"&gt;
    &lt;Say&gt;Now hit some buttons!&lt;/Say&gt;
  &lt;/Gather&gt;
  &lt;Say voice="man"&gt;Awesome! Thanks!&lt;/Say&gt;
  &lt;Hangup/&gt;
&lt;/Response&gt;
</pre>

