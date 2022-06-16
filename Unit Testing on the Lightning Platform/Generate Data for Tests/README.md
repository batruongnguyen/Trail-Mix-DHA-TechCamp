# **[Generate Data for Tests](https://trailhead.salesforce.com/content/learn/modules/unit-testing-on-the-lightning-platform/generate-data-for-tests)**

## Create tests that load data from a CSV in @testSetup method

Create and run a test class that uses a @testSetup method.

**To get started**

    - Download this CSV file.
    - Upload the CSV file as a static resource named Mock_Data.

- Create a new Test Class named myDataGenerationTests
- Create a @testSetup method that loads the Mock_Data csv into Accounts
- Create a test method that asserts the presence of 15 accounts

### Test Class
*myDataGenerationTests.apxc*
```java
@isTest
private class myDataGenerationTests {
  @testSetup
  static void loadTestDataFromStaticResource(){
	List<sObject> accounts = Test.loadData(Account.SObjectType, 'Mock_Data');
  }
  @isTest static void testLoadAccountsFromStaticResource() {
    List<Account> accts = [SELECT ID FROM Account];
    system.assert(accts.size() == 15, 'expected 15 accounts');
  }
}
```

