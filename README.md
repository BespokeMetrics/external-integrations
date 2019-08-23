# Integration Repo

 This repository serves as the hub for all integration pieces for third party.

# Python Consumer

 ## Getting Started
`Docker` is required for running the local `RabbitMQ` server and the connector app. `Docker` can be either run natively like OSX native `Docker` desktop or via using the `Docker-Machine`. 

## Dev Setup

Run `source setup_consumers.sh` for running the RabbitMQ server and the python consumer.
Run `sh teardown_consumer.sh` for shutting down everything.
This setup is provided as a guideline to mimic how the connection to the amqp server
in local environment.

## Local vs Remote RabbitMQ Server

The docker environment varibles are set to local RabbitMQ server. 

RabbitMQ connection parameters can be changed by setting the varibles in the file
`/consumers/python/config/config.py`. For example:

```
def setUp(self):
    os.environ.update({
        "RABBIT_USER": "client",
        "RABBIT_PASS": "guest",
        "RABBIT_HOST": "client",
        "RABBIT_PORT": "5672",
        "RABBIT_VHOST": "/client",
        "RABBIT_QUEUE": "integration",
        "RABBIT_ROUTING": "client.prequal",
        "RABBIT_EXCHANGE": "integration"
    })
```

`RABBIT_ROUTING` refers to the routing key that binds the exchange to the consumer queue.
This will define what kind of data that subscribers are subscribing to. 

For a remote consumer, these variables will need to be set to remote host.
We will provide the clients with specific login infromation to connect to their
respective virtual host.

Deciding on whether to use docker container or not depends solely on the 
discretion of the end user.

## Connection Types

This app allows connecting to rabbit mq server using either the blocking or asynchronous 
connection. By default, the docker setup uses asynchronous connection however the user
can choose to use blocking connection also. 

To use blocking connection, the python command looks like the following:

`python3 -m consumers.python.consume`

To use asynchronous connection, the python command looks like the following:

`python3 -m consumer.python.consume_async`

The benefits of using either one depends on the use case. If the consumer application is 
based on asynchronous architecture then the asynchronous connection provides a non blocking
way of connecting to the rabbit server. On the other hand, for a simple non asynchronous design
the blocking connection will suffice.

## Data Payloads

# Submission Data
Submission data refers to business, finance and health safety data. Following is the example:

```
{
    "tax_number": 123456789,
    "files": ["finance_file.txt", "finance_file2.pdf"],
    "status": "in_review",
    "access_granted": "true",
    "sub_name": "Subcontractor",
    "event": "submission.finance",
    "created_at": 156729456 ## UTC TIME
}
```

`sub_name` refers to subcontractor name.

# Prequalification Data
```
{
    "single_contract_limit": 1234515698.0,
    "aggregate_contract_limit": 12315867912.0,
    "expiry": 156729456,
    "date_assigned": 156729456,
    "assigned_by": "gc@email.com",
    "tax_number": 12315627128,
    "sub_name": "Subcontractor",
    "event": "prequalification.post",
    "created_at": 156729456 ## UTC TIME
}

```

Here expiry timestamp is the unix timestamp in seconds.

# Score Data
This event is published when Q score is updated:
```
{
    "q_score": 1.3,
    "prequalification": {},
    "sub_name": "subcontractor",
    "event": "score.update",
    "created_at": 156729456 ## UTC TIME
}
```
Here the prequalification refers to the prequalification data containing 
all the fields in `Prequalification Data` section except the `event` field.
q_score field refers to the calculated Q score for respective subcontractor.

# New Registration Data (Common to all subscribers)
Example Payload:
```
{
	"email": "david@football.com",
	"first_name": "David ",
	"last_name": "Beckham",
	"title": "Midfielder",
	"phone": "4444444444",
	"locale": "en",
	"additional_users": [],
	"primary_user": {
		"first_name": "David",
		"last_name": "Beckham",
		"email": "david@football.com",
		"phone": "4444444444",
		"title": "Midfielder"
	},
	"created_by": {
		"first_name": "Bruce",
		"last_name": "Wayne",
		"email": "wayne@email.com",
		"phone": "2223339999",
		"title": "CEO"
	},
	"office_locations": [{
		"city": "Toronto",
		"postal_zip": "M5S4A1",
		"province_state": "ON",
		"address": "55 York ",
		"country": "CA",
		"primary": true
	}],
	"size": 1,
	"revenue": 10000000,
	"name": "Beckingham",
	"nationality": "ca",
	"trades": {
		"primary": "Process Interconnections",
		"additional": ""
	},
	"type": "sub",
	"self_registered": true,
	"office": {},
	"tax_number": "123214516416",
	"event": "registration.new_user"
}
```
Here `type: sub` refers to subcontractor type of the new registrant.

# Company Specific Data Updates
This data is delivered when a new sub updates or creates new company specific information.
There are two types of `company.update` that is provided. First is when the company changes information such as trades/ address etc. The second one is when a company changes data authorization. The second update is only visible if the company has authorized the GC to receive this notification.

Example 1: Company Information
```{
	"name": "Wayne Enterprise",
	"tax_number": "445566872",
	"office_locations": [{
		"city": "Toronto",
		"address": "55 York St",
		"country": "CA",
		"primary": true,
		"postal_zip": "M5S4A1",
		"province_state": "ON"
	}],
	"email": "wayne@email.com",
	"phone": "2223334444",
	"trades": {
		"primary": " 01 00 00 - General Requirements",
		"additional": [" 00 00 00 - Procurement and Contracting Requirements", " 03 00 00 - Concrete", " 04 00 00 - Masonry", " 06 00 00 - Wood, Plastics, and Composites", " 13 00 00 - Special Construction"]
	},
	"event": "company.update",
	"updated_at": 1565900004 ## UTC TIME
}
```

Example 2: Company Authorization
```
{
	"name": "Wayne Enterprise",
	"tax_number": "445566872",
	"office_locations": [{
		"city": "Toronto",
		"address": "55 York St",
		"country": "CA",
		"primary": true,
		"postal_zip": "M9A4Y1",
		"province_state": "ON"
	}],
	"email": "wayne@email.com",
	"phone": "2223334444",
	"event": "company.update",
	"updated_at": "2019-08-15T19:02:15+00:00",
	"authorization_updates": {
		"health_data_updated": 1565900004, #UTC TIME
		"finance_data_updated": 1565900004, #UTC TIME
		"business_data_updated": 1565900004, #UTC TIME
		"health_data_authorized": false,
		"finance_data_authorized": false,
		"business_data_authorized": true
	}
}
```

## Unit Testing

There is one unit test suite for checking exceptions for RabbitConsumer class.
Run `python -m consumers.python.test.consumer_tests`.

## Integration Testing

Integration testing requires an active RabbitMQ connection. One way to achieve that
is to test it using the local RabbitMQ server. Currently, the integration test
connection parameters are set to local rabbit host.
