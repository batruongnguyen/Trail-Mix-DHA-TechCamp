# **[Write Permisstion-Based Tests](https://trailhead.salesforce.com/content/learn/modules/unit-testing-on-the-lightning-platform/permission-based-tests)**

## Create a permission-based test

Create a Custom User profile in your Trailhead Playground and give it View All permission on Private_object__c records. Write a positive permission test demonstrating that users with the Custom User profile can view records that they don’t own.

**To get started**

    - In Setup, clone the Custom: Support Profile profile and name it ‘Custom User’.
    - Give the Custom User profile View All permissions on the custom object Private_Object__c. (Listed as Private Objects in the user interface)
- Create a new test class named PositivePermission_tests
- Write a positive permission unit test showing that users with the - Custom User profile can access Private_Object__c records that they do not own
- Execute your unit tests and ensure that they all pass

### **How to create a Custom User profile in your Trailhead Playground**
1. From Setup enter `User Management Settings` in the *Quick Find* box, and select *User Management Settings*.
2. Find Custom User (Ctril F and type **Custom User**)
3. Find Private Object (Ctril F and type **Private Object**)
6. Click to `View All`
### Test Class
*PositivePermission_tests.apxc*
```java
@isTest
private class PositivePermission_tests {
  @testSetup
  static void testSetup(){
    Account a = TestFactory.getAccount('View For You!', true);
    Private_Object__c po = new Private_Object__c(account__c = a.id, notes__c = 'foo');
    insert po;
  }
  @isTest
  static void PermissionSetTest_Positive() {
    User u = new User(
      ProfileId = [SELECT Id FROM Profile WHERE Name = 'Custom User'].Id,
      LastName = 'last',
      Email = 'Cpt.Awesome@awesomesauce.com',
      UserName = 'Cpt.Awesome.' + DateTime.now().getTime() + '@awesomesauce.com',
      Alias = 'alias',
      TimeZoneSidKey = 'America/Los_Angeles',
      EmailEncodingKey = 'UTF-8',
      LanguageLocaleKey = 'en_US',
      LocaleSidKey = 'en_US'
    );
    System.runAs(u){
      Private_Object__c[] pos;
      Test.startTest();
        pos = [SELECT Id, Account__c, notes__c FROM Private_Object__c];
      Test.stopTest();
      system.assert(pos.size()!= 0, 'a user with the permission set can see respective records');
    }
  }
}
```