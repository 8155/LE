"Documentation for external developers" draft / sample file:

# Leveris Digital Platform

![Leveris logo](https://res-5.cloudinary.com/crunchbase-production/image/upload/c_lpad,h_120,w_120,f_auto,b_white,q_auto:eco/v1460129189/ctlloub15lbsya2mxwtv.png)


## LDB intro
Leveris Digital Bank (LDB) is advanced core banking platform in the market. Modern, modular, non-legacy banking system architected to address the key challenges banks face today. Featuring:
- Modular Design - possible to replace or remove modules without affecting the others
- Service Oriented Architecture (SOA) - possible to use only services clients want
- Open APIs

There is some generic information valid throughout whole API documentation:
`{ general info }`

Two channels are supported - WEB and MOBILE, and in both cases users are required to proceed through the User Activation process (aka onboarding) process in order to use the LDB application.

## Onboarding
- introduces new users to the application, collects their personal data, confirms the identity
- consists of a set of steps new users are required to perform before being allowed into the very system
- is configurable by defined process parametrisation or steps definition via .yml files [link]()
- partners are free to set up their own FE design and appearance but the step constraints must be met for every step

## STEP types
There are three different step types recognied:
- StepType: "**FORM**"
	-  generic, used for collecting and saving data
	- e.g. collecting basic info, i.e. first name, last name, e-mail address, etc.
	- can be configured to collect multiple data out of predefined set of types
- StepType: "**AUTH**"
	- generic, used for specific data verification
	- can be configured to collect multiple data out of predefined set of types
	- can be configured by scenarios
- StepType: "**CUSTOM**"
	- custom, can be added to the parametrisation optionally
	- can be wildly configured, inner logic is fully programmable
	- can be tailored for external partners or any specific service use
	- e.g. Identity verification supported by Jumio 3rd party service 

## Onboarding process start

To start the Onboarding process a specific api is called:
```
/api/private/process/processes/!startProcess
```

- starts new process using `channel` and `definitionCode` parameters
- returns unique identification of started process using `idProcess` parameter
- parameter options, return codes, and complete API description to be find here [link]()
- parameters are not sent as a part of URL (URII params), instead an Object is created and parameters are sent as a payload within its body properties

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

### API definition: /processes/!startProcess
The exact and up-to-date description can be find at
 [https://doc.ffc.internal/api/mw-gen-user-activation-ib/Process-ib/1.0/Process-ib.raml.html](https://doc.ffc.internal/api/mw-gen-user-activation-ib/Process-ib/1.0/Process-ib.raml.html) 

#### API description example:


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
---

## Onboarding process definition

In order for client to know what steps are configured for given onboarding process definition a `getDefinitionByProcessId` is called:
```
/api/private/process/processes/{idProcess}/!getDefinitionByProcessId
```

- uses `idProcess` parameter
- retrieves valid process definition using configured `steps` (list of steps for this process) and `processCode` (unique code of process definition, e.g. "ONBOARDING"), based on `idProcess` parameter returned by `!startProcess` 
- this process definition can be used for showing progress of the ongoing process for example
- parameter options, return codes, and complete API description to be found here [link]()
- all steps of the process definition are to be found here [link]()

API call example:
```
Call:
	api/private/processes/12345/!getDefinitionByProcessId

Body:
	{
	  "idProcess" : "12345"
	} 

Response:
	{
	  "processCode" : "CLIENT_ONBOARDING"
	  "steps" : ""  //TBC
		0: {code: "BASIC_INFO", description: "Basic information"}
		1: {code: "BASIC_INFO_COUNTRY", description: "Residency"}
		2: {code: "VERIFY_EMAIL", description: "Verify your email"}
		3: {code: "VERIFY_PHONE", description: "Verify your phone"}
		4: {code: "SMS_CODE", description: "Enter the code from SMS"}
		5: {code: "SET_PASSWORD", description: "Set your password"}
		6: {code: "OPEN_BANK_ACC", description: "Open bank account"}
		7: {code: "SET_DEVICE_ID", description: "Scan your finger"}
		8: {code: "DEVICE_CREDENTIALS", description: "Save device credentials and obtain on-device secret"}
		9: {code: "IDV_JUMIO", description: "Identity document verification"}
		10: {code: "JUMIO_POA", description: "Address extraction"}
		11: {code: "TAC", description: "Terms & Conditions"}
		12: {code: "ADDRESS_APPROVAL", description: "Address approval"}
		13: {code: "KYC", description: "KYC"}
	}
```




### API definition: /processes/{idProcess}/!getDefinitionByProcessId

The exact and up-to-date description can be find at
 [https://doc.ffc.internal/api/mw-gen-user-activation-ib/Process-ib/1.0/Process-ib.raml.html](https://doc.ffc.internal/api/mw-gen-user-activation-ib/Process-ib/1.0/Process-ib.raml.html) 

#### Api description example:

DESCRIPTION:
Retrieves valid process definition based on the process id

REQUEST:
-  URI Parameters:
	- **idProcess:**  _string_ _(required)_  | maxLength: 100
	Process identification

RESPONSE:									(TBC)
- 200: StepResult.FINISH if no next step is needed
	-  Body: StepResult.FINISH if no next step is needed
		- **Media type**: application/json
		 **Type**:  _object ProcessDefinitionResponse_
	- Object properties:
	    -   **steps:**  _array of_  _ProcessStep_  _(required)_  
	        list of steps for this process
	        
	    -   **processCode:**  _string_ _(required)_  | maxLength: 100  
	        Unique code of process definition like ONBOARDING
- 400: API entry validation error
- 410: Entity was not found
- 500: Server error
---


## Onboarding process execution

In order to proceed trough the onboarding process steps, the `execute` api must be called:
```
/api/private/process/processes/{idProcess}/steps/!execute
```
- `!execute` can be called when all the required actions of the currents step are met
- is called to push the entire process to the next step, which is defined by process definition (example: [link]())
- uses `idProcess`,  `mediaType`, and `party` parameters to get the `status` and `nextStep` in response


#### API definition: /processes/!execute
DESCRIPTION:   TBC

 - StepResult.FINISH if no next step is needed

REQUEST:
-   URI Parameters
	- **idProcess:**  _string_ _(required)_  | maxLength: 100
	Process identification
- Body
	- **Media type**: application/json
	-  **Type**:  _object StepExecutionRequest_
-  Object properties:
	-  **party:**  _object_  _processPartyTypes.Party_

RESPONSE:
- Body
	- **Media type:** application/json
	- **Type:** object StepExecutionResponse
- Object properties:
	- **status:** string one of "NEW", "PENDING", "DONE", "FAILED" (required)
NEW - Process is created, but any step has not been accomplish yet | PENDING - Process is valid, at least one step has been already accomplish but whole process has not finished yet | DONE - Process is in termination status and process finish successfully | FAILED - Process is in termination status and proces finish unsucessfully
- **nextStep**: object userProcessTypes.ProcessStep
Return definition of next step if status PENDING. IF DONE or FAIL then return nothing

	Example:
	```
	{
	  "code": "BASIC_INFO_STEP",
	  "stepType": "FORM",
	  "private": false,
	  "label": "Register",
	  "formProperties": [
	     {
	       "fieldName": "party.individualName.firstName",
	       "mandatory": false
	     },
	     {
	       "fieldName": "party.individualName.surName",
	       "mandatory": true
	     }
	   ]
	}
	```
-	**tokens:** object authTypes.LoginTokens (required)
JWT token for acctivation
---

## Onboarding process recommended steps/flow:

- one of the entities involved and handled in the oboarding process is the Persona or Party.
- to create a new party we need at least one `BASIC_INFO` step, where user fills in the e-mail address
	Step example:
	```
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
    }
    ```
- output of this is a new entity (we call it Party) that:
	- exists already
	- has got the basic information on it (e.g. first name, last name, e-mail address)
	- has got the user account created, existing and assigned
		- such user account is not *Active* at that moment yet, since it hasn't got all the necessary credentials
		
  


## Onboarding process parameterization example
- yml files structure is used
- for new clients following process parameterization example is recommended:
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

## Onboarding process API calls sequence example
- for new clients and suggested parameterization the API calls sequence may look like this example:
```yml
FE action (start button)
  !list
  !startProcess
  !getDefinitionByProcessId
FE action (basic info, name, surname, e-mail, button)
  !execute
  !getAuthScenarioDefinition
  !startAuthProcess
FE action (e-mail confirmation URL)
  !getDefinitionByProcessId
  !getAuthScenarioDefinition
  !validateAuthProcessStep
  !execute
FE action (country of residence, button)
  !execute
FE action (phone, button)
  !execute
  !getAuthScenarioDefinition
  !startAuthProcess
FE action (phone validation, sms, button)
  !validateAuthProcessStep
  !execute 
FE action (password, button)
  !execute
  !searchProcesses
  !getDefinitionByProcessId
FE action (Verify identity button)
  !initiateNetverifyRedirect
FE action (POI, submit button)
  !getDefinitionByProcessId
  !getJumioScanStatus
  !setScanProcessStatus
  !execute
FE action (POA, button)
  !acquisitionsProcess?initiationToken=123
  !getDefinitionByProcessId
  !execute
FE action (TaC, button)
  !execute
  !getDefinitionByProcessId
  !searchProcesses
  !list
  !getDefinitionByProcessId
  !execute
//onboarding sample ends here, next rec. step might be !getLoginScenario or product offer for example
```

## SECURITY
- https, auth, tokens, etc info follows:
- APIs are secured by tokens, and those must be sent with every API call
- security tokens can be re-issued or prolonged, as described here: [link]()

## API description
- list and complete description for all the API needed for the onboarding part of the process 
```
System of documentation template:
 -1- process/business info
 -2- mw api: name
 -3- description
 -4- definition
 -5- example
```
## Revision list
2018/12 - first concepts of the document
2018/01 - creating value, adding information, work in progress

---
## Remarks
//formats, layout, and final information structure to be polished by a member of the target audience (e.g. FE developer)


