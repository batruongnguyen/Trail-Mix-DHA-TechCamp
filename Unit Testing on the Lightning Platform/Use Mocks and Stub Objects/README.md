# **[Use Mocks and Stub Objects](https://trailhead.salesforce.com/content/learn/modules/unit-testing-on-the-lightning-platform/mock-stub-objects)**

## Write tests using mocks and stubs

Not every call to a third party service succeeds. So, we need to test both success and failure responses. Write a unit test to cover the ExternalSearch classes’ handling of response codes higher than 200.

- Add a unit test method to ExternalSearch_tests.cls that uses the HTTPMockFactory class to return a 500 response
- Make sure you have 100% code coverage on the ExternalSearch class

*ExternalSearch_Tests.apxc*
```java
@isTest
private class ExternalSearch_Tests {
  @isTest
  static void test_method_one() {
    HttpMockFactory mock = new HttpMockFactory(200, 'OK', 'I found it!', new Map<String,String>());
    Test.setMock(HttpCalloutMock.class, mock);
    String result;
    Test.startTest();
      result = ExternalSearch.googleIt('epic search');
    Test.stopTest();
    system.assertEquals('I found it!', result);
  }
   @isTest static void test_method_two() {
    HttpMockFactory mock = new HttpMockFactory(500, 'Internal Server Error', 'server issue!', new Map<String,String>());
    Test.setMock(HttpCalloutMock.class, mock);
    String result;
    Test.startTest();
      result = ExternalSearch.googleIt('server issue');
    Test.stopTest();
    system.assertEquals('server issue!', result); 
  }

  @isTest static void test_method_three() {
    HttpMockFactory mock = new HttpMockFactory(404, 'Server not found', 'server not found!', new Map<String,String>());
    Test.setMock(HttpCalloutMock.class, mock);
    String result;
    Test.startTest();
         try {
              result = ExternalSearch.googleIt('server not found');
         }
         catch (ExternalSearch.ExternalSearchException exc){
             system.assert(exc.getMessage().equalsIgnoreCase('Did not recieve 200/500 status code'));
         }
     
    Test.stopTest();    
  }
}
```

*ExternalSearch.apxc*
```java
public class ExternalSearch {
	public class ExternalSearchException extends Exception{}
  
  public static string googleIt(String query){
    Http h = new Http();
    HttpRequest req = new HttpRequest();
    req.setEndpoint('https://www.google.com?q='+query);
    req.setMethod('GET');
    HttpResponse res = h.send(req);
    if(res.getStatusCode() != 200){
      throw new ExternalSearchException('Did not recieve a 200 status code: ' + res.getStatusCode());
    }
    return res.getBody();
  }
  
}
```