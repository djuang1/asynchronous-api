# Asynchronous Request-Reply API

## Overview

Most APIs are designed to respond quickly to a request, generally 100ms or less. But there are cases where the backend systems will take longer than the expected reply. So the question is how to design a RESTful API to address these use cases? 

## Solution

Asynchronous APIs, or HTTP Polling, is the approach to address these use cases. From a design standpoint, it's a combonation of standard HTTP headers and status codes. HTTP 202 Accepted being the main code that lets the user know that the request has been received and is being processed.
