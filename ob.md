# Leveris Digital Bank - onboarding

This is documentation of the services of LDB onboarding process.
There are some rules that are valid throughout whole API.

## LDB intro (header TBD)
Leveris Digital Bank (LDB) is advanced core banking platform in the market. Modern, modular, non-legacy banking system architected to address the key challenges banks face today. Featuring:
- Modular Design - possible to replace or remove modules without affecting the others
- Service Oriented Architecture (SOA) - possible to use only services clients want
- Open APIs - publicly available

Two channels of operation are provided (WEB client and IB smartphone client) and in both cases users are required to proceed through an onboarding process in order to use the LDB application.

## Onboarding (header TBD)
- introduces new users to the application, collects their personal data, confirms the identity
- consists of a set of steps new users are required to perform before being allowed into the very system
- these steps are configurable by defined process parametrisation [link]()
- customer is free to set up their own FE design and appearance but the client definition must be met for every step

### Onboarding process
To start the Onboarding process a specific api is called:
```
/api/private/process/processes/!startProcess
```
- starts new process using `channel` and `definitionCode` parameters
- returns unique identification of started process using `idProcess` parameter
- parameter options, return codes, and complete API description to be find here [link]()
- parameters are not sent as a standard part of URL, instead an Object is created and parameters are sent as it's Payload 

Example:
```
Call:
api/private/process/processes/!startProcess

Body:
{
  "channel": "WEB",
  "definitionCode": "CLIENT_ONBOARDING"
}

Response:
{
"idProcess": "123456"
}
```

---

In order for client to know how to continue with the onboarding process, the process ID is used for next api call:

```
/api/private/process/processes/{idProcess}/!getDefinitionByProcessId
```
- uses `idProcess` parameter
- based on the process id retrieves valid process definition using parameter `steps` (list of steps for this process) and `processCode` (unique code of process definition, like "ONBOARDING" for example)
- parameter options, return codes, and complete API description to be find here [link]()
- all steps of the process definition are to be find here [link]()


## WIP
In case all the predefined requirements of current step are met, process is pushed to the next step by calling `!execute` api:
```
path/path/!execute
```
- requests X
- returns Y
- link to complete API description



there are ## steps
Pomoci basic info do systemu dostaneme zakladni neoverene udaje
nasleduje overovani
k tomu je potreba... atd

Example request:
```
curl -X GET https://api.airbank.cz/openapi/banking/v1/transactions?sort=category&limit=10&after=15
```

Example response pagingInfo:

```javascript
"pagingInfo": {
    "nextPage": "transactions?sort=category&limit=10&after=25"
    "itemsPerPage": 10,
}
```


## API description TBD


### /processes/!startProcess
DESCRIPTION:
Starts new process based on the channel and process code parameters

REQUEST:
- Body:
	- **Media type**: application/json
	- **Type**:  _object StartProcessRequest_
- Object properties:
	- **definitionCode:**  _string_ _(required)_  | maxLength: 100
	Identification of process definition
	- **channel:**  _string_ _(required)_  | maxLength: 100
	Identification of the channel used for onboarding


RESPONSE:
- 200: Return new instance of process
	- Body:
		- **Media type**: application/json
		- **Type**:  _object StartProcessResponse_
	- Object properties:
		- **idProcess:**  _string_  _(required)_ | maxLength: 100
		Unique identification of process.
- 400: API entry validation error
	- Body:
		- **Media type**: application/json
		- **Type**:  _object ErrorHolder_
	- Object properties:
	- **errors:**  _any_ 
	List of errors. Empty in case of unexpected or infrastructure errors.
	-   **idCall:**  _string_  
	    Optional call identifier for easier troubleshooting (will be present only for unexpected errors).
	-   **resource:**  _string_  
	    The first known source of the error.

## Process Parameterization TBD
```yml
api: "{be-del-user-activation}/user-activation/processes/definition/!param"
method: "post"
okReturnValues: [200]
records:
  "data":
    - {
        "processCode": "CLIENT_ONBOARDING",
        "channel": "WEB",
        "priority": 20,
        "proofOfAddressMinValidity": 6,
        "proofOfAddressFixedCountry": true,
        "steps": [
            {
                "businessStage": "REGISTRATION",
                "code": "BASIC_INFO",
                "formProperties": [
                     {
                        "fieldName": "party.individualName.firstName",
                        "mandatory": true
                     },
                     {
                        "fieldName": "party.individualName.surName",
                        "mandatory": true
                     },
                     {
                        "fieldName": "party.contacts.primaryEmail",
                        "mandatory": true
                     }
                ],
                "label": "VERIFY_EMAIL",
                "private": false,
                "stepType": "FORM",
                "setClientNumber": true,
                "backendStep": false
            },
            {
                "businessStage": "REGISTRATION",
                "code": "AUTH_EMAIL",
                "label": "VERIFY_EMAIL",
                "private": false,
                "stepType": "AUTH",
                "authOperation": "ACTIVATE",
                "backendStep": false
            },
            {
                "businessStage": "REGISTRATION",
                "code": "BASIC_INFO_COUNTRY",
                "formProperties": [
                     {
                        "fieldName": "party.country",
                        "mandatory": true
                     }
                ],
                "label": "BASIC_INFO_COUNTRY",
                "private": false,
                "stepType": "FORM",
                "backendStep": false
            },
            {
                "businessStage": "REGISTRATION",
                "code": "PHONE",
                "formProperties": [
                     {
                        "fieldName": "party.contacts.primaryPhone",
                        "mandatory": true
                     }
                ],
                "label": "VERIFY_PHONE",
                "private": false,
                "stepType": "FORM",
                "backendStep": false
            },
            {
                "businessStage": "REGISTRATION",
                "code": "AUTH_SMS",
                "label": "SMS_CODE",
                "private": false,
                "stepType": "AUTH",
                "authOperation": "ACTIVATE_STEP_1",
                "backendStep": false
            },
            {
                "businessStage": "REGISTRATION",
                "code": "PASSWORD",
                "label": "SET_PASSWORD",
                "private": false,
                "activateIdentity": true,
                "stepType": "FORM",
                "formProperties": [
                     {
                        "fieldName": "party.authDetails.password",
                        "mandatory": true
                     }
                ],
                "backendStep": false
            },
            {
                "businessStage": "DUE_DILIGENCE",
                "code": "JUMIO",
                "label": "IDV_JUMIO",
                "private": true,
                "stepType": "CUSTOM",
                "backendStep": false
            },
            {
                "businessStage": "DUE_DILIGENCE",
                "code": "JUMIO_POA",
                "label": "IDV_JUMIO_POA",
                "private": true,
                "stepType": "CUSTOM",
                "backendStep": false
            },
            {
                "businessStage": "DUE_DILIGENCE",
                "code": "TAC",
                "label": "TAC",
                "private": true,
                "stepType": "CUSTOM",
                "backendStep": false
            },
            {
                "businessStage": "DUE_DILIGENCE",
                "code": "ADDRESS_APPROVAL",
                "label": "ADDRESS_APPROVAL",
                "private": true,
                "stepType": "CUSTOM",
                "backendStep": true
            },
            {
                "businessStage": "DUE_DILIGENCE",
                "code": "KYC",
                "label": "KYC",
                "private": true,
                "stepType": "CUSTOM",
                "backendStep": true
            },
            {
                "businessStage": "OPEN_BANK_ACC",
                "code": "APP_ACC",
                "label": "OPEN_BANK_ACC",
                "private": true,
                "stepType": "APPLICATION",
                "backendStep": false,
                "makeClient": true
            }
          ]
      }
```

## Generic rules and information
- lorem ipsum
- lorem ipsum
- lorem ipsum
- lorem ipsum
