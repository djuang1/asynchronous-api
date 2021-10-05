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
