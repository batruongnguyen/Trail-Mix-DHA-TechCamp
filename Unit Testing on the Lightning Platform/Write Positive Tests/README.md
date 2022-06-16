# **[Write Positive Tests](https://trailhead.salesforce.com/content/learn/modules/unit-testing-on-the-lightning-platform/positive-tests)**

## Write a positive unit test

Open AccountWrapper_Tests.cls and add a test method that positively tests the isHighPriority() method.
- Add a test method to the AccountWrapper_Tests.cls that positively tests the isHighPriority() method
- Be sure to use @testSetup methods to produce your test data
- Run your unit tests and make sure you have at least 85% code coverage on the AccountWrapper.cls

*AccountWrapper_Tests.apxc*
```java
@isTest
private class AccountWrapper_Tests {
  @testSetup
  static void loadTestData(){
    List<Account> accounts = (List<Account>) Test.loadData(Account.SObjectType, 'accountData');
    List<Opportunity> opps = new List<Opportunity>();
    for(Account a : accounts){
      opps.addAll(TestFactory.generateOppsForAccount(a.id, 1000.00, 5));
    }
    insert opps;
  }
  @isTest
  static void testPositiveRoundedAveragePrice() {
    List<AccountWrapper> accounts = new List<AccountWrapper>();
    for(Account a : [SELECT ID, Name FROM ACCOUNT]){
      accounts.add(new AccountWrapper(a));
    }
    // sanity check asserting that we have opportunities before executing our tested method.
    List<Opportunity> sanityCheckListOfOpps = [SELECT ID FROM Opportunity];
    System.assert(sanityCheckListOfOpps.size() > 0, 'You need an opportunity to continue');
    Test.startTest();
      for(AccountWrapper a : accounts){
        System.assertEquals(a.getRoundedAvgPriceOfOpps(), 1000.00, 'Expected to get 1000.00');
      }
    Test.stopTest();
  }
  @isTest static void testPositiveIsHighPriority() {
        List<AccountWrapper> accounts = new List<AccountWrapper>();
        for(Account a : [SELECT ID, Name FROM ACCOUNT]){
            accounts.add(new AccountWrapper(a));
        }
       // sanity check asserting that we have opportunities before executing our tested method.
       // back grab the ammount field so we can update
        List<Opportunity> sanityCheckListOfOpps = [SELECT ID, Amount FROM Opportunity];
        System.assert(sanityCheckListOfOpps.size() > 0, 'You need an opportunity to continue');
	for (Opportunity opp : sanityCheckListOfOpps ) {
		opp.Amount = 100000;
	}
	//update the list of opportunities with the new value
	update sanityCheckListOfOpps ;
        Test.startTest();
        for(AccountWrapper a : accounts){
            System.assertEquals(a.isHighPriority(), false, 'Expected isHighPriority to return false');
        }
        Test.stopTest();
    }
  @isTest
  static void testNegativeAccountWrapperRoundedOpps(){
    List<Account> accts = [SELECT Id FROM Account];
    List<Opportunity> opps = [SELECT ID, Amount FROM Opportunity WHERE accountId in :accts];
    List<AccountWrapper> wrappers = new List<AccountWrapper>();
    for(Opportunity o : opps){
      o.amount = 0;
    }
    update opps;
    for(Account a : accts){
      wrappers.add(new AccountWrapper(a));
    }
    Test.startTest();
      List<Boolean> exceptions = new List<Boolean>();
      for(AccountWrapper a : wrappers){
        try{
          a.getRoundedAvgPriceOfOpps();
        } catch (AccountWrapper.AWException awe){
          if(awe.getMessage().equalsIgnoreCase('no won opportunities')){
            exceptions.add(true);
          }
        }
      }
    Test.stopTest();
    system.assertNotEquals(null, exceptions, 'expected at least one exception to have occured');
    for(Boolean b : exceptions){
      system.assert(b, 'Account should have thrown an exception');
    }
  }
}
```

*AccountWrapper.apxc*
```java
public with sharing class AccountWrapper {
  public class AWException extends exception{}

  Account thisAccount;

  public AccountWrapper(Account startWith){
    thisAccount = startWith;
  }

  public decimal getRoundedAvgPriceOfOpps(){
    AggregateResult[] ar = [SELECT AVG(Amount) 
                            FROM Opportunity 
                            WHERE accountId = :thisAccount.id];
                            system.debug(ar);
    Decimal average = (Decimal) ar[0].get('expr0'); 
    Long modulus = Math.mod(Integer.valueOf(average), 1000);
    Decimal returnValue = (modulus >= 500) ? (average + 1000) - modulus : average - modulus;
    if(returnValue <= 0) { 
      throw new AWException('No won Opportunities');
    }
    return returnValue;
  }

  public Boolean isHighPriority(){
    if(getRoundedAvgPriceOfOpps() > 100000.00){
      return true;
    }
    return false;
  }
}
```