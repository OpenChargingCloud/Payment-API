# Payment API

**DRAFT**

Revision: 1.0

## Contributors

| Roll    | Responsible                          | Company     |
|---------|--------------------------------------|-------------|
| Author  | Elias Böhmer , chargecloud GmbH (EB) | chargecloud |


## Revision overview

Changes from the previous version are marked change tracking.

| Revision | Content                                          | Date          |
|----------|--------------------------------------------------|---------------|
| 0.1      | First draft of Elias Böhmer (chargecloud GmbH)   | 28.02.2023/EB | 

###	Versioning of the document

The version number of this document is structured as follows:

**Major Version:**
Changes at this point mean that the format has changed fundamentally. Changes have been made to the format that are no longer compatible with previous versions.

**Minor:**
The format was supplemented new fields or the values of a field were added (changes to the format. Backward compatibility is given).

**Revision:**
Change to the documentation. No change to the format. Increasing the revision does not change the format itself. The documentation is adapted or improved.

Example: 1.0.2
1 = Major
0 = Minor
2 = Revision


# General

## Challenges

For the connection of credit card terminals to the charging infrastructure, as will be mandatory in Germany from 01 July 2023, an interface is required between the credit card terminals or the associated CPMS systems of the providers and the charging stations or the charge point management systems (CPMS) used for the billing of charging transactions.

## Targets

In order to keep the development effort low for all companies involved and to enable a quick and simple smooth implementation, the interface presented here is to be submitted as a standard. A uniform interface simplifies the connection for all hardware and software manufacturers in the field of charging infrastructure.

## Users

- Hardware manufacturers of electric charging stations
- Hardware manufacturers of payment terminals (especially credit card terminals)
- Software provider of CPMS

## Which types of jobs do we want to include?
1. When I connect a credit card terminal to an electric charging station,
2. I want to be able to use a uniform interface
3. so that I don't have to develop it again for each hardware manufacturer or for each CPMS.

## Documentation

### Glossary / List of Abbreviations
- CPMS (Charge Point Management System)
- EV Driver (Electric car driver)
- EVSE (Electric Vehicle Supply Equipment)
- OCPP (Open Charge Point Protocol, see Protocols, Home - Open Charge Alliance)

### Participants
- EVSE / Charging Station
- CPMS
- Terminal Backend
- Acquirer
- Payment Terminal
- EV Driver

## Relevant web services
The following section describes in more detail the relevant messages that need to be sent between Terminal Backend and CPMS.

### Logo

![](/Sequence_Diagrams/Logo.png "Logo")


**Description**
This web service can be used to retrieve the specified logo of a client in Base64 format.

**Definition of endpoint structure**

`{base_url}/location/logo`

| Method | Description                                  |
|--------|----------------------------------------------|
| GET    | Retrieve a logo as it is stored in the CPMS. |


**Request Parameter**

| Parameter | Datatype | Required | Description |
|-----------|----------|----------|-------------|
| n/a       |          |          |             | 

**Response Data**

| Parameter            | Meaning                                                                                                                             |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| ``errorCode``        | Returns an errorCode in case of an error; otherwise `null`.                                                                         |
| ``messageLocalized`` | In the event of an error, returns the error message in the display language.                                                        |
| ``dataType``         | Returns `undefined`" in case of error; otherwise `array`.                                                                           |
| ``data``             | Returns `null` in case of an error; otherwise `logo` and `modificationDate`.                                                        |
| ``logo``             | Not transmitted in case of error; otherwise a logo in Base64 format.                                                                |
| ``modificationDate`` | Not transmitted in case of error; otherwise the time of the last modification of the logo file in the format `YYYY-MM-DD hh:mm:ss`. |

**Potential error codes**

| Error code                | Meaning                 |
|---------------------------|-------------------------|
| `no_emobility_logo_found` | No logo could be found. | 

### DirectPaymentData

![](/Sequence_Diagrams/DirectPaymentData.png "Logo")

**Description**

This web service serves as the initial entry point of the Direct Payment Service. It outputs all relevant information about the charging point as well as relevant information about the client to which the charging point belongs. This web service searches for the EVSE ID across all clients. The charging point is identified by the parameter evseId .

**Definition of endpoint structure**

``{base_url}/location/{evseId}``

| Method | Description                                                    |
|--------|----------------------------------------------------------------|
| GET    | Retrieve the Direct Payment Data as it is stored in the CPMS.  |


**Request Parameter**

| Parameter     | Datatype | Required | Description                                                                                                                                                                                      |
|---------------|----------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ``evseId``    | string   | yes      | This parameter identifies a charging point by using the transferred EVSE ID (see https://github.com/hubject/oicp/blob/master/OICP-2.3/OICP%202.3%20EMP/03_EMP_Data_Types.asciidoc#EvseIDType).   |


**Response Data**

| Parameter            | Meaning                                                                                                                                                |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| ``errorCode``        | Returns an errorCode in case of an error; otherwise null.                                                                                              |
| ``messageLocalized`` | In case of an error, returns the error message in the display language.                                                                                |
| ``dataType``         | Returns "undefined" in case of error; otherwise "array".                                                                                               |
| ``data``             | Returns null in case of an error; otherwise a JSON object with all relevant information about the charging point and the associated client is output.  |

**Potential error codes**

| Error code                       | Meaning                                                 |
|----------------------------------|---------------------------------------------------------|
| ``charge_point_not_found``       | Charging point could not be found                       |
| ``direct_payment_not_available`` | Ad-hoc access is not available for this charging point  |

### Tariff

![](/Sequence_Diagrams/Tariff.png "Tariff")

**Description**

This web service can be used to display the tariff valid for the customer at a charging point. The charging point is identified by the parameter evseId. This web service must be requested before each charging process so that the correct charging customer is always informed about the current price. It must be ensured that the customer sees the actual price to be paid and can also confirm this. The confirmed price for this customer may change during the charging process.Definition of endpoint structure

`{base_url}/tariff/{evseId}`

| Method | Description                                                                |
|--------|----------------------------------------------------------------------------|
| GET    | Retrieve the Pricing Data of the Chargepoint as it is stored in the CPMS.  |

**Request Parameter**

| Parameter   | Datatype | Required | Description                                    |
|-------------|----------|----------|------------------------------------------------|
| ``evseId``  | string   | yes      | This parameter identifies the charging point.  |

**Response Data**

| Parameter            | Meaning                                                                                                                 |
|----------------------|-------------------------------------------------------------------------------------------------------------------------|
| ``errorCode``        | Returns an errorCode in case of an error; otherwise null.                                                               |
| ``messageLocalized`` | In case of an error, returns the error message in the display language.                                                 |
| ``dataType``         | Returns "undefined" in case of error; otherwise "array".                                                                |
| ``data``             | Returns null in case of an error; otherwise an array with the valid price structure at the identified charging point.   |


**Potential error codes**

| Error code                       | Meaning                                                                         |
|----------------------------------|---------------------------------------------------------------------------------|
| ``charge_point_not_found``       | Charging point could not be found                                               |
| ``direct_payment_not_available`` | Ad-hoc access is not available for this charging point                          |
| ``direct_payment_is_free``       | Charging at this charging point is free of charge, so no tariff can be issued.  |
| ``undefined_error``              | An unknown error has occurred.                                                  |


### TransactionStart

![](/Sequence_Diagrams/TransactionStart.png "TransactionStart")


**Description**

This web service can be used to start a loading process. The started loading process is registered as a payment instance in the CPMS. This web service is called as soon as the pre-authorisation of the desired charging amount has been successfully completed and the customer wants to charge.

**Definition of endpoint structure**

`{base_url}/transaction/start`

| Method | Description                                     |
|--------|-------------------------------------------------|
| POST   | Sending a command to start a charging process.  |



**Request Parameter**

| Parameter         | Datatype | Required | Description                                                                                                                                                      |
|-------------------|----------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ``uuid``          | string   | yes      | This unique identifier is specified by the terminal backend. It can be used to query the status of the charging process as well as to end the charging process.  |
| ``chargePointId`` | string   | yes      | This ID is used to identify the charging point at which the charging process takes place.                                                                        |
| ``emailAddress``  | string   | yes      | The transferred e-mail address will be used for sending the invoice after the charging process is completed.                                                     |
| ``uuidMerchant``  | string   | no       | This parameter specifies which configured merchant is responsible for processing the payment.                                                                    |
| ``callbackUrl``   | string   | no       | This parameter can be used to pass a URL to the CPMS, to which it can transmit all relevant information of the charging process after it has been completed.     |


**Response Data**

| Parameter            | Meaning                                                                 |
|----------------------|-------------------------------------------------------------------------|
| ``errorCode``        | Returns an errorCode in case of an error; otherwise ``null``.           |
| ``messageLocalized`` | In case of an error, returns the error message in the display language. |
| ``dataType``         | Returns ``undefined``                                                   |
| ``data``             | Returns ``null``                                                        |

**Potential error codes**

| Error code                                  | Meaning                                                                    |
|---------------------------------------------|----------------------------------------------------------------------------|
| ``charge_point_not_found``                  | The charging point could not be found                                      |
| ``direct_payment_not_available``            | Ad-hoc access is not available for this charging point                     |
| ``instance_with_uuid_does_exist_already``   | Transferred payment reference (uuid) already exists in the system          |
| ``merchant_not_found``                      | No payment provider merchant could be found with the given merchant UUID.  |
| ``authenticator_not_found``                 | No ad-hoc access key could be found for this charging point                |
| ``remote_start_transaction_timeout``        | Timeout for remote start transaction                                       |
| ``remote_start_transaction_occupied``       | Charging point is already occupied                                         |
| ``remote_start_transaction_rejected``       | Remote Start Transaction was rejected                                      |
| ``remote_start_transaction_unknown_error``  | Unknown error with Remote Start Transaction                                |

### Transaction

![](/Sequence_Diagrams/Transaction.png "Logo")

**Description**

This web service can be used to output the status and details of a charging process that has already been started. The charging process is identified by the uuid parameter. The terminal can use this web service to show the requested transaction data on the display.

**Definition of endpoint structure**

``{base_url}/transaction/{uuid}``

| Method | Description                                                 |
|--------|-------------------------------------------------------------|
| GET    | Requests the status of the charging process from the CPMS.  |


**Request Parameter**

| Parameter | Datatype | Required | Description                                                                                                                                                    |
|-----------|----------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ``uuid``  | string   | yes      | This parameter identifies a charging process that has already been started. The identifier must be the same as the one used when starting a charging process.  |


**Response Data**

| Parameter            | Meaning                                                                                                                            |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------|
| ``errorCode``        | Returns an errorCode in case of an error; otherwise ``null``.                                                                      |
| ``messageLocalized`` | In the event of an error, displays the error message in the display language.                                                      |
| ``dataType``         | Returns ``undefined`` in case of error; otherwise ``array``.                                                                       |
| ``data``             | Returns nothing in case of an error; otherwise a JSON object with all relevant information about the charging process is returned. |



**Potential error codes**

| Error code                | Meaning                                                                   |
|---------------------------|---------------------------------------------------------------------------|
| ``transaction_not_found`` | No charging process could be found with the specified payment reference.  |
| ``undefined_error``       | An unknown error has occurred.                                            |


### TransactionStop


![](/Sequence_Diagrams/TransactionStop.png "TransactionStop")

**Description**

This web service can be used to end a charging process that has been started. The charging process is identified by the parameter uuid.

**Definition of endpoint structure*+

``{base_url}/transaction/stop``

| Method | Description                                   |
|--------|-----------------------------------------------|
| POST   | Sending a request to end a charging process.  |


**Request Parameter**

| Parameter   | Datatype | Required | Description                                                                                                                                                             |
|-------------|----------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ``uuid``    | string   | yes      | This parameter is used to identify a charging process that has already been started. The identifier must be the same as the one used when starting a charging process.  |


**Response Data**

| Parameter            | Meaning                                                                       |
|----------------------|-------------------------------------------------------------------------------|
| ``errorCode``        | Returns an errorCode in case of an error; otherwise ``null``.                 |
| ``messageLocalized`` | In the event of an error, displays the error message in the display language. |
| ``dataType``         | Returns ``undefined``.                                                        |
| ``data``             | Returns ``null``.                                                             |



**Potential error codes**

| Error code                                 | Meaning                                                                                 |
|--------------------------------------------|-----------------------------------------------------------------------------------------|
| ``payment_instance_not_found``             | No matching payment instance with the given payment reference could be found.           |
| ``charge_point_not_found``                 | Charging point could not be found                                                       |
| ``adhoc_stop_not_supported``               | The charging point does not support the stopping of a Direct Payment charging process.  |
| ``transaction_not_found``                  | Charging process could not be found                                                     |
| ``remote_stop_transaction_timeout``        | Timeout for Remote Stop Transaction                                                     |
| ``remote_stop_transaction_rejected``       | Remote Stop Transaction was rejected                                                    |
| ``remote_stop_transaction_unknown_error``  | Unknown error with Remote Stop Transaction                                              |


### TransactionCallback

![](/Sequence_Diagrams/TransactionCallback.png "TransactionCallback")

**Description**

This web service transmits the status of the completed charging process to the terminal backend. The charging process is identified by the parameter uuid. This web service is only triggered if a callback URL has been transmitted to the CPMS by the TransactionStart web service.

**Definition of endpoint structure**

``{callback_url}``

| Method | Description                                                                           |
|--------|---------------------------------------------------------------------------------------|
| POST   | The status of the completed charging process is transmitted to the terminal backend.  |

**Request Parameter**

| Parameter      | Datatype | Required | Description                                                                                                                      |
|----------------|----------|----------|----------------------------------------------------------------------------------------------------------------------------------|
| ``uuid``       | string   | yes      | This parameter identifies a charging process. The identifier must be the same as the one used when starting a charging process.  |
| ``data``       | Object   | yes      | Returns a JSON object with all relevant information about the charging process.                                                  |

**Response Data**

| Parameter            | Meaning                                                                        |
|----------------------|--------------------------------------------------------------------------------|
| ``errorCode``        | Returns an errorCode in case of an error; otherwise null.                      |
| ``messageLocalized`` | In the event of an error, displays the error message in the display language.  |
| ``dataType``         | Returns “undefined".                                                           |
| ``data``             | Returns null.                                                                  |


**Potential error codes**

| Error code                | Meaning                              |
|---------------------------|--------------------------------------|
| ``transaction_not_found`` | Charging process could not be found  |
| ``undefined_error``       | An unknown error has occurred.       |


### PaymentInstructions

![](/Sequence_Diagrams/PaymentInstructions.png "PaymentInstructions")

**Description**

This web service outputs all uncollected and posted payment instructions (payment instructions from merchant bank accounts).

**Definition of endpoint structure**

``{base_url}/instructions``

| Method | Description                                                          |
|--------|----------------------------------------------------------------------|
| GET    | Request for payment orders that have not been collected and booked.  |


**Request Parameter**

| Parameter | Datatype | Required | Description  |
|-----------|----------|----------|--------------|
| n/a       |          |          |              |


**Response Data**

| Parameter            | Meaning                                                                                                 |
|----------------------|---------------------------------------------------------------------------------------------------------|
| ``errorCode``        | Returns an errorCode in case of an error; otherwise null.                                               |
| ``messageLocalized`` | In the event of an error, displays the error message in the display language.                           |
| ``dataType``         | Returns "undefined" in case of error; otherwise "array".                                                |
| ``data``             | In data, an export file with several payment instructions in a JSON schema is returned for each entry.  |



**Potential error codes**

| Error code            | Meaning                         |
|-----------------------|---------------------------------|
| ``undefined_error``   | An unknown error has occurred.  |


### PaymentInstructionsResults

![](/Sequence_Diagrams/PaymentInstructionsResult.png "Logo")

**Description**

This web service is used to return the result of payment instruction executions back to the CPMS. This web service expects an array of JSON encoded strings.

**Definition of endpoint structure**

``{base_url}/instructions``

| Method | Description                       |
|--------|-----------------------------------|
| POST   | Sends processed payments to CPMS  |

**Request Parameter**

| Parameter               | Datatype | Required | Description                                                                                                                   |
|-------------------------|----------|----------|-------------------------------------------------------------------------------------------------------------------------------|
| ``instructionResults``  | array    | yes      | This parameter contains an array of JSON encoded strings. Each string represents a Direct Payment Instruction Result object.  |

**Response Data**

| Parameter            | Meaning                                                                       |
|----------------------|-------------------------------------------------------------------------------|
| ``errorCode``        | Returns an errorCode in case of an error; otherwise ``null``.                 |
| ``messageLocalized`` | In the event of an error, displays the error message in the display language. |
| ``dataType``         | Returns ``undefined`` .                                                       |
| ``data``             | Returns ``null`` .                                                            |


**Potential error codes**

| Error code | Meaning  |
|------------|----------|
| n/a        |          |



### Example: entire process of setting up a charging station as well as a charging process


![](/Sequence_Diagrams/FullSequenceDiagram.png "Full Sequence Diagram")
 