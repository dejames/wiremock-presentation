#WIREMOCK

---
##Agenda

* Introduction to wiremock
* Why wiremock?
  * Limitation with provider-mocks in flight-service
  * Problems of using air-spoofer-service 
* How is it used in flight-service?

---
##Features

* Running Wiremock
* Request matching
* Record and Playback
* Response transformation

---

##Running WireMock

1. in junit mode

2. as a standalone jar

mappings are available at http://localhost:9960/__admin/mappings 

+++

in junit mode

```java
@org.junit.Rule
public WireMockRule wireMockRule = new WireMockRule(WireMockConfiguration.options()
                                    .port(9521)
                                    .usingFilesUnderDirectory("../fss-acceptance-mocks/src/main/resources/stubs"));

```

+++
standalone mode

1. can be started directly using JAR
```java
java -jar wiremock-standalone-2.7.1.jar
```
2. can be started programmatically
```java
   public static void main(String[] args) {
       int wireMockPort = retrieveWireMockPort(args);
       int providerMocksPort = retrieveProviderMocksPort(args);
       String stubsDirectory = retrieveStubsDirectory(args);
       startWireMock(wireMockPort, providerMocksPort, stubsDirectory);
   }
    
   public static WireMockServer startWireMock(int wireMockPort, int providerMocksPort, String stubsDirectory) {
       LOGGER.info("Start mocked flight-service external service mock server on port {}", wireMockPort);
       WireMockServer wireMockServer = WireMockInitializer.initialize(wireMockPort, providerMocksPort, stubsDirectory);
       wireMockServer.start();
       LOGGER.info("Mocked flight-service external service mock server successfully started on port {}", wireMockPort);
       return wireMockServer;
   }
```

---

##Request Matching

folders mappings and __files are created automatically once you start wiremock
1. mappings    : contains the data to match request
2. __files     : contains the response to be sent back

+++

![mocks and stubs](assets/fileDirectory.PNG)

mapping : 
```json
{
  "priority": 1,
  "request": {
    "customMatcher": {
      "name": "fast-infoset-request-matcher",
      "parameters": {
        "url": "/provider-mocks/sabre/get-details",
        "method": "POST",
        "headers": {
          "Transaction-Id": {
            "equalTo": "SABRE RT pricing with a BUSINESS fare"
          }
        },
        "bodyPatterns": [
        ]
      }
    }
  },
  "response": {
    "status": 200,
    "bodyFileName": "pricing/sabre_pricing_cabin_class/body-sabre-getdetails.xml",
    "headers": {
      "Server": "Apache-Coyote/1.1",
      "User-Agent": "Apache CXF 3.0.4",
      "Content-Type": "text/xml;charset=UTF-8",
      "Accept": "*/*",
      "Accept-Encoding": "gzip;q=1.0, identity; q=0.5, *;q=0"
    }
  }
}
```

---

##Record and Playback

1. new     : using web interface http://localhost:8080/__admin/recorder
2. legacy  : java -jar wiremock-standalone-2.7.1.jar --proxy-all="http://search.twitter.com" --record-mappings

---

##Response transformation

transforming certain fields in mocked response

+++

```java
    @Override
    public class ResponseDateTransformer extends ResponseTransformer {
    
        @Override
        public Response transform(Request request, Response response, FileSource files, Parameters parameters) {
            try {
                List<LocalDate> requestDates = RequestDateReader.extractDateForEachBound(request);
    
                if (!CollectionUtils.isEmpty(requestDates)) {
                    Optional<ResponseDateEditor> responseDateEditor = ResponseDateEditorSelector.selectResponseDateEditor(response);
                    if (responseDateEditor.isPresent()) {
                        return responseDateEditor.get().editResponseDate(response, requestDates);
                    }
                }
            } catch (Exception ignored) {
            }
            return response;
        }
    
        @Override
        public String getName() {
            return "response-date-transformer";
        }
    }
    
    WireMockConfiguration wireMockConfiguration = new WireMockConfiguration();
    wireMockConfiguration.extensions(ResponseDateTransformer.class);
```
@[1-24](transformimg dates in response to those in request)
@[26-27] (adding transformer to wiremock configuration)

---

##How Egencia flight-service uses wiremock

---

##References

1. [wiremock in flight-service!](https://stash.sea.corp.expecn.com/projects/EGES/repos/flight-service/browse/fss-acceptance/pom.xml#529)
2. [wiremock docs!](http://wiremock.org/docs/)
---

##Thank you!


