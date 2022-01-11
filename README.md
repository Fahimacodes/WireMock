# WireMock
Documentation of research and development utilizing Wire Mock to simulate the request/response events of API’s to automate testing

In this project we used the Wire Mock standalone JAR and Zoom API 
  http://wiremock.org/docs/download-and-installation/ 
  http://wiremock.org/docs/running-standalone/
  
  The 3 main concepts that were beneficial in speeding up testing were Stubbing, Request Matching and Response Templating.
  
### Stubbing 
- to return canned HTTP responses for requests matching criteria. This can be done in ways, we have explored using C# and JSON API.

1) C# POST to create mapping:

           static async Task HttpClientPost()
          {
              using (var httpClient = new HttpClient())
              {
                  using (var request = new HttpRequestMessage(new HttpMethod("POST"), "<http://localhost:8080/__admin/mappings"))>
                  {
                      request.Content = new StringContent("{\"request\": {\"method\":\"GET\",\"url\": \"/test1\"},\"response\": {\"bodyFileName\": \"get_archive_event.xml\"}}");
                      request.Content.Headers.ContentType = MediaTypeHeaderValue.Parse("text/plain");

                      var response = await httpClient.SendAsync(request);
                  }
              }
          }

2) JSON file copied to “mappings” folder:

        {"request":
            {
              "urlPath": "/v2/archive_files",
              "method": "GET",
              "queryParameters" : { 		  
              "from" : {
                    "matches" : "^(\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2})Z$"
                  },
              "to" : {
                    "matches" : "^(\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2})Z$"
                  },
              "page_size" : {
                    "matches" : "15"
                  },
                  "query_date_type" : {
                    "matches" : "archive_complete_time"
                  }
              }
            }, "response": {"bodyFileName": "archive_files.xml"}}
    
In “response” the “bodyFileName” is located under “__files” directory of where Wire Mock is running. Body contents can be input directly instead like “ “body” : “Hey” too.

### Request Matching
- to match requests to stubs and verification queries using URL, Headers, Request Body Contents and more. We explored match via Query Parameters.

Endpoint = v2/archive_files - query parameters after “?” should be matched (see code snippet 2) line 34.
Code snipped 2) is mapping for Archive Files requests --> https://api.zoom.us/v2/archive_files?from=2021-08-25T09:05:15Z&to=2021-09-01T09:05:15Z&page_size=15&query_date_type=archive_complete_time

### Response Templating 
- to configure response headers and bodies, as well as proxy URLs, that can optionally be rendered using Handlebars templates. This enables attributes of the request to be used in generating the response. We explored this in dynamically setting start_time in our response body as it is queried.

Alter start_time/end_time/duration in a response mapping for each request. 
Example -
"start_time":"{{now}}" ← Use current time (UTC timezone)
"start_time":"{{date (parseDate request.query.from) offset='+4 hours'}}"  <-- Use time 4 hours ahead of time (UTC timezone)
