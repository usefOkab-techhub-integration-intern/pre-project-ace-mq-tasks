# Pre-Project ACE MQ Tasks

This document provides step-by-step guidance for various tasks related to IBM App Connect Enterprise (ACE) and MQ.

## Task 1: Integration Node Creation and Security

1. **Create an Integration Node with Username, Password, and VaultKey**
2. **Change HTTP and Debug Ports**
3. **Create a Message Flow Based on Input Queue**
   - Create a message flow that processes the following input bodies:
     ```json
     { "queue": 1 }
     ```
     ```json
     { "queue": 2 }
     ```
   - Based on the value of `queue`, send a different static message in response.
   - **Dynamic Update**: To change the static message during runtime without redeployment, store the messages in a database or a configurable service, allowing updates without needing to redeploy the message flow.

## Task 2: File Splitting and Record Transfer

1. **Message Flow to Read a File and Split Records**
   - Create a message flow that reads a file containing multiple records.
   - Split the file into separate records and write each record to a new file.
   - Example record structure:
     ```json
     { "record": "data1" }
     { "record": "data2" }
     ```

## Task 3: Database Interaction with Error Handling

1. **CRUD Operations Using a Message Flow**
   - Create a message flow that performs a CRUD operation on a database.
   
2. **Throw an Exception During the Request**
   - Simulate an exception by sending a request that will cause an error (e.g., an invalid database query).

3. **Exception Formatting**
   - Capture the exception from the ExceptionList and format it as:
     ```json
     { "ErrorMessage": "", "ErrorDescription": "" }
     ```

4. **Test Different Exception Scenarios**
   - Simulate various types of exceptions to test your error handling, such as connection failures, invalid queries, or missing data.

## Task 4: Event Monitoring and Logging

1. **Enable Event Monitoring on the Flow from Task 3**
   - Monitor the flow at the start and end to capture relevant events.

2. **Save Event Data to a Database**
   - Create another application to consume the monitored events and store the event information in a database.

## Task 5: WSDL Generation Documentation

1. **WSDL Generation for SOAP Request and Response**
   - Document how to generate a WSDL file for the following SOAP request and response:

   **Request:**
   ```xml
   <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ipn="http://www.mdsl.com/IPNCardAuth/">
      <soapenv:Header/>
      <soapenv:Body>
         <ipn:CardAuthRequest>
            <requestID>1234567890</requestID>
            <timeStamp>2024-01-11 08:35:15.387</timeStamp>
            <cardNumber>5211821822530500</cardNumber>
            <PIN>1ECCA61D0D746744</PIN>
         </ipn:CardAuthRequest>
      </soapenv:Body>
   </soapenv:Envelope>
   ```

 **Response:**
   ```xml
   <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
   <soapenv:Body>
      <a:CardAuthResponse xmlns:a="http://www.mdsl.com/IPNCardAuth/">
         <requestID>1234567890</requestID>
         <timeStamp>2024-01-11 08:35:15.387</timeStamp>
         <Result>
            <resultCode>0000</resultCode>
            <resultDesc>Success</resultDesc>
         </Result>
      </a:CardAuthResponse>
   </soapenv:Body>
</soapenv:Envelope>
```

## Task 6: HTTPS Configuration

1. Write a documentation about how to expose https apis from the app connect and how to invoke https api.
