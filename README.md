# redis-pusher

redis-pusher is a Node.js application that subscribes to channels on Redis,
consumes messages published there, and send them as Push Notification via
Apple Push Notification Service (APNS).

1. Workflow

	a) Events: Sit and listen for messages published on the Redis subscribed
	   channel(s).

	b) Processing: Convert each published message to the APNS message format
	   and dispatch them to the APNS server.

	c) Feedback: Periodically connect to the APNS Feedback server, and receive a
	   list of devices that have persistent delivery failures so that you can
	   mark them as bad and refrain from sending new push notifications to them.

2. Sharding

	You can split the workload across multiple instances.
	To do that, configure `production.redis.channels` in your `private/config.js`.

3. Failover redundancy

	You may have multipe instances subscribed to the same channel(s) simultaneously.
	To do that, set `production.redis.failoverEnabled` to `true` in your
	`private/config.js`.

4. What redis-pusher does not attempt to do:

	It does not process any of the APNS feedback messages. This is something
	specific to your scenario. For example, your scenario might involve an SQL
	table containing all registered devices (id, device_token, etc), so you
	could mark them as bad, or simply delete them. It's entirely up to you!

- - -

### How can I test it?

##### Install the required tools

1. [Node.js](http://nodejs.org/)
2. [Redis](http://redis.io/)

##### Clone the repository

  $ git clone git@github.com:jweyrich/redis-pusher.git

##### Install the dependencies

	$ cd redis-pusher
	$ npm install

##### Test locally

###### Configure

a) Copy your APNS certificates and keys to the private
   directory that resides within the project's directory.

	$ cp /path/to/your/apns_development.p12 \
		/path/to/your/apns_production.p12 \
		private/

b) Copy the example configuration file from the project's directory
to the private directory.

	$ cp example.config.js private/config.js

c) Edit your private configuration file according to your needs. Change
   the values below to match your development APNS certificate's password.

	development.apns.gateway.options.passphrase
	development.apns.feedback.options.passphrase

###### Run it

	$ node pusher.js development &
	$ redis-cli
	redis> publish development:push:ios '{ "identifier": 1, "device_token": "<device_token>", "expires": 300, "badge": 1, "sound": "default", "alert": "You have a new message" }'

##### Message format

	message {
		identifier: [number] -- Unique identifier
		device_token: [string]
		expires: [number] -- Seconds from now
		badge: [number]
		sound: [string]
		alert: [string]
		payload: [object]
	}