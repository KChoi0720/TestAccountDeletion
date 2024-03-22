// Account Trigger

trigger AccountDeletion on Account (before delete) {
    // stop delete account if relate opportunity
    for (Account a : [SELECT Id FROM Account
                      WHERE Id IN (SELECT AccountId FROM Opportunity) AND
                      Id IN :Trigger.old]) {
         					Trigger.oldMap.get(a.Id).addError(
             				'Cannot delete account with related opportunities.');
     }
 }


//Prepare DataFactory

@isTest public class TestDataFactory {
    public static List<Account> createAccountsWithOpps(Integer numAccts, Integer numOppsPerAcct) {
         List<Account> accts = new List<Account>();        
         for(Integer i=0;i<numAccts;i++) {//numAccts:创建的Account数量
             Account a = new Account(Name='TestAccount' + i);
             accts.add(a);
         }
         insert accts;
         
         List<Opportunity> opps = new List<Opportunity>();
         for (Integer j=0;j<numAccts;j++) {
             Account acct = accts[j];
            //review every Account, and create a related opportunity
            for (Integer k=0;k<numOppsPerAcct;k++) {
                 opps.add(new Opportunity(Name=acct.Name + ' Opportunity ' + k,
                                        StageName='Prospecting',
                                        CloseDate=System.today().addMonths(1),
                                        AccountId=acct.Id));
             }
         }
        insert opps;
        return accts;
    }
} 

@isTest private class TestAccountDeletion {
     @isTest static void TestDeleteAccountWithOneOpportunity() {

        // create an Account, and relate to an Opportunity
        Account[] accts = TestDataFactory.createAccountsWithOpps(1,1);
        //Start test
		Test.startTest();
        Database.DeleteResult result = Database.delete(accts[0], false);
        Test.stopTest();
        //check verification
        //should be prohibitted to delete (already relate to opportunity)
        //should return error warning: prohibitted to delete
		System.assert(!result.isSuccess());
        System.assert(result.getErrors().size() > 0);
        System.assertEquals('Cannot delete Account which is related to Opportunity',
                             result.getErrors()[0].getMessage());
    }

    @isTest static void TestDeleteAccountWithNoOpportunities() {
        //test initialization
        //create an Account, not relate to Opportunity
        Account[] accts = TestDataFactory.createAccountsWithOpps(1,0);

        //start test
		Test.startTest();
        Database.DeleteResult result = Database.delete(accts[0], false);
        Test.stopTest();
        //delete successfully
        System.assert(result.isSuccess());
    }

    @isTest static void TestDeleteBulkAccountsWithOneOpportunity() {
        //initialize test data
        //create 200 Accounts, create one Opportunity
        Account[] accts = TestDataFactory.createAccountsWithOpps(200,1);

        //start test
		Test.startTest();
        Database.DeleteResult[] results = Database.delete(accts, false);
        Test.stopTest();

        //verify result
        //bulkly delete, some accounts cannot be deleted, need to roll back
        //get a wrong return
        for(Database.DeleteResult dr : results) {
            System.assert(!dr.isSuccess());
            System.assert(dr.getErrors().size() > 0);
            System.assertEquals('Cannot delete account with related opportunities',
                                 dr.getErrors()[0].getMessage());
        }
    }

    @isTest static void TestDeleteBulkAccountsWithNoOpportunities() {
        //initialization
        //create bulky data not related to Opportunities
        Account[] accts = TestDataFactory.createAccountsWithOpps(200,0);

        //start test
        Test.startTest();
        Database.DeleteResult[] results = Database.delete(accts, false);
        Test.stopTest();
        //bulkly deleted successfully
		for(Database.DeleteResult dr : results) {
            System.assert(dr.isSuccess());
        }
    }
}