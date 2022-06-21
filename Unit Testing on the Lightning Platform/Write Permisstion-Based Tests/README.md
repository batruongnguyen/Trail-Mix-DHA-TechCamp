# **[Write Permisstion-Based Tests](https://trailhead.salesforce.com/content/learn/modules/unit-testing-on-the-lightning-platform/permission-based-tests)**

## Create a permission-based test

Create a Custom User profile in your Trailhead Playground and give it View All permission on Private_object\_\_c records. Write a positive permission test demonstrating that users with the Custom User profile can view records that they don’t own.

**To get started**

    - In Setup, clone the Custom: Support Profile profile and name it ‘Custom User’.
    - Give the Custom User profile View All permissions on the custom object Private_Object__c. (Listed as Private Objects in the user interface)

- Create a new test class named PositivePermission_tests
- Write a positive permission unit test showing that users with the - Custom User profile can access Private_Object\_\_c records that they do not own
- Execute your unit tests and ensure that they all pass

### **How to clone a Custom User profile in your Trailhead Playground**

1. From Setup enter `Profiles` in the _Quick Find_ box, and select _Profiles_
2. Find Custom: Support Profile and click that profile
3. Click `clone`, enter name `Custom Profile`
4. When your profile already exists, select it and click `edit`
5. Find _private objects_ and click `View all`
6. `Save`

### Test Class

_PositivePermission_tests.apxc_

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
