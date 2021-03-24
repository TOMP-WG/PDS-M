# Personal Data Storage API for Mobility (PDS-M)

This API describes the data that is needed by Transport Operators (TO) and MaaS Providers (MP). It can be used alongside the TOMP-API (https://github.com/TOMP-WG/TOMP-API/).  

It has a few endpoints: 
- for requesting personal information for routing
- for requesting personal information for booking
- for requesting personal information for support
- for posting trip journals

## Requesting personal information
This feature has 2 modes: for routing purposes (MP side), it is (in most cases) not needed to get real card numbers, names, etc. Just giving meta data (like kinds of (license) cards, age etc) will provide enough information to plan and route (Zero Knowledge Proof).  

After the routing, booking will require extra information for validation, like card/membership numbers, passport numbers, etc. etc.

After a trip it should be logged to determine preferences.
  
## Examples 
### Personal information for routing ###
This request is followed by a few possible examples.
``` 
POST /routing/{personal-token}/query
[
  {
    "index": 0,
    "label": "AGE",
    "operator": "GE",
    "value": "18"
  }, 
  {
    "index": 1,
    "label": "GENDER",
    "operator": "VALUE"
  }, 
  {
    "index": 2,
    "label": "ABILITIES",
    "operator": "VALUE"
  }, 
  {
    "index": 3,
    "label": "LOCATION",
    "operator": "COUNT",
    "value": "52.3483,8.3942"
  }, 
  {
    "index": 4,
    "label": "HOME",
    "operator": "VALUE"
  }, 
  {
    "index": 5,
    "label": "DRIVER_LICENSE",
    "operator": "FOR",
    "value": "car"
  },
  {
    "index": 6,
    "label": "MEMBERSHIP",
    "operator": "FOR",
    "value": "NS businesscard"
  }
]
```
### Example response 1: ###
This response is for a male, above 18 years, in a wheelchair. Somewhat familiar at the specified location (in the logs he has been seen 18 times last year). He doesn't want to expose his home location to the requesting party, and has no driver license. He also doesn't want to expose the fact he has a NS businesscard.
```
[ { "index": 0, "answer": true }
, { "index": 1, "answer": "MALE" }
, { "index": 2, "answer": [ { "category": "HR", "number": "01" } ] }
, { "index": 3, "answer": "18" }
, { "index": 4, "answer": "hidden" }
, { "index": 5, "answer": false }
, { "index": 6, "answer": "hidden" }
]
```
### Example response 2: ###
This response if for a adult female, hasn't been before at the requested location, is allowing to show her home address, doesn't have a driver license and has a NS businesscard.
```
[ { "index": 0, "answer": true }
, { "index": 1, "answer": "FEMALE" }
, { "index": 2, "answer": [] }
, { "index": 3, "answer": "0" }
, { "index": 4, "answer": "56.343243,19.329849" }
, { "index": 5, "answer": false }
, { "index": 6, "answer": true }
]
```

### Personal information for booking ###
In case of a booking this is a valid request (followed again by some valid examples):
``` 
POST /booking/{personal-token}/query
[
  {
    "index": 0,
    "label": "AGE",
    "operator": "VALUE"
  }, 
  {
    "index": 1,
    "label": "GENDER",
    "operator": "VALUE"
  }, 
  {
    "index": 2,
    "label": "ABILITIES",
    "operator": "VALUE"
  }, 
  {
    "index": 3,
    "label": "DRIVER_LICENSE",
    "operator": "FOR",
    "value": "car",
    "required": true
  }
]
```
### Example response 1: ###
Male, 33 years old, in a wheel chair. Has a driver license, no 3490230342923.
```
[
  { "index": 0, "answer": 33 },
  { "index": 1, "answer": "MALE" },
  { "index": 2, "answer": [ { "category": "HR", "number": "01" } ] },
  { "index": 3, "answer": "3490230342923", "proof": "https://someurl.to/mydocument.pdf" }
]
```
### Example response 2: ###
Female, 39 years old, doesn't want to expose her abilities nor her driver license number.
```
[
  { "index": 0, "answer": 39 },
  { "index": 1, "answer": "FEMALE" },
  { "index": 2, "answer": "hidden" },
  { "index": 3, "answer": "hidden" }
]
```
It might be obvious that if a required field returns 'hidden', it might block the continuation of processes.
