# Asynchronous Request-Reply API

## Overview

Most APIs are designed to respond quickly to a request, generally 100ms or less. But there are cases where the backend systems will take longer than the expected reply. So the question is how to design a RESTful API to address these use cases? 

## Solution

Asynchronous APIs, or HTTP Polling, is the approach to address these use cases. From a design standpoint, it's a combonation of standard HTTP headers and status codes. HTTP 202 Accepted being the main code that lets the user know that the request has been received and is being processed.


<img src="https://github.com/djuang1/asynchronous-api/blob/main/docs/async-request.png?raw=true" width="600px">


## Project

This repository includes an example MuleSoft application that shows how to implement the Asychronous API pattern. The ObjectStore is used to maintain the state of each job in the backend. In order to complete the processing of the backend job, just call the resource `/tasks/{id}` in order to move it to the next state.


```
#%RAML 1.0
title: Asynchronous API
version: 1.0.0
mediaType: 
- application/json

types:
  job: 
    properties:
      status:
        enum: [IN PROGRESS, CANCELLED, FAILED, SUCCESS]
      start: string
      end: string
                
/resources:       
  get:
    description: Get list of tasks
    responses:
      200:
        body:
          application/json:
            example: |
              {
                "tasks":[
                    {
                      "id":"21303",
                      "href":"/resources/21303"
                    },
                    {
                      "id":"21304",
                      "href":"/resources/21304"
                    }
                ]
              }
  post:
    description: Create a resource task
    responses:
      202:
        description: 
        headers:
          Location:
            example: /resources/21304
        body:
          example: |
            { 
              "id": "21304",
              "href": "/resources/21304"
            }            
  /{id}:
    get:
      description: Get a resource
      responses:
        200:
          body:
            type: job
            example: {status: "IN PROGRESS",start: "01-01-2013 10:50:45",end: ""}           
        303:
          body:
            type: job
            example: {status: "SUCCESS",start: "01-01-2013 10:50:45",end: "01-01-2014 10:50:45"}
        500:
          body:
            type: job
            example: {status: "FAILED",start: "01-01-2013 10:50:45",end: "01-01-2014 10:50:45"}
    delete:
      responses:
        200:
          description: Delete a resource
          body:
            type: job
            example: {status: "CANCELLED",start: "01-01-2013 10:50:45",end: "01-01-2014 10:50:45"}

/tasks/{id}:
  put:
    responses:
      200:
        body:
          application/json:
            example: {status: "UPDATED"}
```

## Code Examples

Below are areas in the code that I wanted to call out because it's not well documented.

### Setting Outbound Headers - Location

The HTTP 202 response indicates the location where subsequent requests will check on the status of the job. In order to set the outbound header in the HTTP response, you need to set a variable using the following [code](https://github.com/djuang1/asynchronous-api/blob/main/src/main/mule/asynchronous-api.xml#L235):

````
<set-variable value="#[output application/java --- (vars.outboundHeaders default {}) ++ {&quot;Location&quot;: vars.job.href}]" doc:name="Set Location" doc:id="1e60fa70-b954-46bd-8323-78dc6ced358c" variableName="outboundHeaders" />
````

### DataWeave - Map LinkedHashmap to Array

Another problem I had to solve was how to map a LinkedHashmap of records from the ObjectStore to an array so I could map them in the JSON response. Below is the code that I used:

````
%dw 2.0
output application/json
---
{
  tasks: dw::core::Objects::entrySet(payload) filter (($.key as String) != "task") map { 
  		id: $.key,
		href: "/resources/" ++ $.key
	}
}
````