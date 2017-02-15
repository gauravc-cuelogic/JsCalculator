# FPC


**FPC Health Check - To check server status**</br>
* API End Point</br>
  - <yourDomain>/device/health_check</br>
    * Result:</br>
      - Success Result</br>
        * Response : {"FPC":"Success","Database":"Success","Rouge":"Success"} </br>
        * Status Code : 200 OK</br>
      - Failure Result</br>
        * Response : {"FPC":"Success","Database":"Failure","Rouge":"Success"} </br>
        * Status Code : 404 Not Found</br>
