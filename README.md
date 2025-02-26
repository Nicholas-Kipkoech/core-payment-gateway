# Backend Software Engineer Technical Interview

## Integrations

### 1. INTRODUCTION 3

1.1. Overview 3
1.2. Scope 3
1.3. Terminology & Abbreviations 3
1.3.1. PG 3
1.3.2. CPS 3
1.3.3. API 3
1.3.4. GwRequest 4
1.3.5 .Result 4 2. TASK 5
2.1. Processes 5
2.1.1. Receive Payment Request from Core Payments System (CPS) 5
2.1.2. Validate, Log & Send to 3rd Party PG 5
2.1.3. Receive 3rd Party Callback, Validate, Update Log & Send Result back to CPS 6
2.1.4. Query Payment Status using 3rd Party API 6
2.2. System Architecture 7
2.3. Stack 7
2.3.1. Spring, MariaDB & Kafka 7
2.3.2. Go, MongoDB & Kafka 7 3. CONCLUSION 8

### 4. INTRODUCTION

1.1. Overview
The purpose of this document is to outline all the key technical areas that interviewee must be conversant with for them to join the Tanda technical team. The interview uses a real life system for integration. The system of our choice is The Daraja B2C API. This API provides a great challenge and helps Tanda test real life engineering competencies. Please note that Tanda has an existing Daraja B2C integration already in production and your code will be used for evaluation purposes only.
1.2. Scope
This interview will test:
Event-driven/Message-driven/Non-blocking programing
Java/Go
OAuth 2.0
RESTful APIs
SQL/NoSQL
Scheduling tasks
Src code documentation
Writing tests
1.3. Terminology & Abbreviations
1.3.1. PG
Payment gateway.
1.3.2. CPS
Core payments system. Tanda’s billing, ledgers and wallets management platform.
1.3.3. API
Application programming interface.
1.3.4. GwRequest
This is the request sent from the Core to a 3rd Party integration that holds data about a payment request.IThe json properties to expect
id (uuid) , the transaction id
amount (float), the transaction amount
mobileNumber (MSISDN), E.164 formatted recipient mobile number
1.3.5 .Result
This is the payload that is sent back from the 3rd Party integration to the Core. The json properties expected.
id (uuid), the transaction id
status , the final status
ref (string), the mpesa reference

### 5. TASK

2.1. Processes
2.1.1. Receive Payment Request from Core Payments System (CPS)
Receive payment request (GwRequest) from CPS via Kafka listening to the provided topic.
The payload needs to be validated to ensure it conforms to the specifications of the GwRequest.
All mandatory fields should be present
Data types should be as expected.
2.1.2. Validate, Log & Send to 3rd Party PG
Before sending the request you need to validate the payment request.
The mobileNumber and amount should be present.
The mobileNumber should be a valid kenyan and Safaricom mobile number.
The amount should be a valid amount
The amount should be between KSh 10 and KSh 150,000.

After validating , we need to log the payment request to the database. We need to log enough info from the GwRequest to be able to identify the request from the callback and also build a Result payload. The correct status should be used (Pending).

Next we need to send the request to the 3rd Party PG .
Package the 3rd Party Request
Send the request to the PG
Check the response to ensure request was submitted successfully
Update payment request logon failed
Package Result and send failed response on failed

Incase request fails due to a network issue we need to log it to a pending request table to be used for status query.
2.1.3. Receive 3rd Party Callback, Validate, Update Log & Send Result back to CPS
There should be a callback handler to receive the final result from the PG. On reception of the callback we need to
Validate the payload
Check if the payment exists on the payment request table
Check the status on the callback
Update the payment log with correct status
Delete entry on the pending request log if exists
Package a Result
Send result to Core via kafka
2.1.4. Query Payment Status using 3rd Party API
There should be a recurring task that checks for the statuses of all the payments on our pending log. It should do the following
Pull all the pending logs from the database
Check the status of each request
On success or fail update the payment request log
On success or fail delete from the pending log
On success or fail package Result and send to Core via kafka
If the status is still pending increment the retry count

2.2. System Architecture

2.3. Stack
Please use one of the following tech combinations to complete the task.
2.3.1. Spring, MariaDB & Kafka
Spring boot, Spring WebMVC, Spring Data JPA, Bean validation 2.0, Spring Kafka
MariaDB
Docker
Apache Kafka server, Apache Zookeeper
SQL
2.3.2. Go, MongoDB & Kafka
Go 1.20.x, net/http, Segmetio Kafka client, Go validator v10, etc
Apache Kafka server, Apache Zookeeper
MongoDB
Docker
NoSQL 3. CONCLUSION
We expect that all the source code will be available on a public GitLab repository for review by dedicated engineers from Tanda.
