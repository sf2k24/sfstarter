Here’s a professional, streamlined summary of the Apex code for the Vehicle Service Station App functionalities:

---

### 1. **Aggregate Payments and Service Counts on Account**

```apex
trigger UpdateAccountStats on Service__c (after insert, after update, after delete) {
    Set<Id> accountIds = new Set<Id>();
    for (Service__c service : Trigger.isDelete ? Trigger.old : Trigger.new) {
        if (service.Account__c != null) accountIds.add(service.Account__c);
    }
    
    List<Account> accountsToUpdate = new List<Account>();
    for (AggregateResult result : [SELECT Account__c Id, 
                                         SUM(Payment__c) totalPayments, 
                                         COUNT(Id) totalServices, 
                                         COUNT(CASE WHEN Status__c = 'Rejected' THEN 1 END) rejectedServices 
                                    FROM Service__c 
                                    WHERE Account__c IN :accountIds 
                                    GROUP BY Account__c]) {
        Account acc = new Account(Id = (Id)result.get('Id'));
        acc.Total_Payments__c = (Decimal)result.get('totalPayments');
        acc.Total_Services__c = (Integer)result.get('totalServices');
        acc.Rejected_Services__c = (Integer)result.get('rejectedServices');
        accountsToUpdate.add(acc);
    }
    update accountsToUpdate;
}
```

---

### 2. **Prevent Duplicate Contacts Under the Same Account**

```apex
trigger PreventDuplicateContacts on Contact (before insert, before update) {
    Set<String> uniqueKeys = new Set<String>();
    for (Contact con : Trigger.new) {
        String uniqueKey = con.AccountId + '-' + con.Email;
        if (uniqueKeys.contains(uniqueKey)) con.addError('Duplicate contact under this account is not allowed.');
        uniqueKeys.add(uniqueKey);
    }
}
```

---

### 3. **Calculate Total Amount Spent on Services for Each Contact**

```apex
trigger UpdateContactSpending on Service__c (after insert, after update, after delete) {
    Map<Id, Decimal> contactTotals = new Map<Id, Decimal>();
    for (AggregateResult result : [SELECT Contact__c Id, SUM(Payment__c) totalSpent 
                                   FROM Service__c WHERE Contact__c IN :Trigger.newMap.keySet() 
                                   GROUP BY Contact__c]) {
        contactTotals.put((Id)result.get('Id'), (Decimal)result.get('totalSpent'));
    }
    
    List<Contact> contactsToUpdate = new List<Contact>();
    for (Id contactId : contactTotals.keySet()) {
        contactsToUpdate.add(new Contact(Id = contactId, Total_Amount_Spent__c = contactTotals.get(contactId)));
    }
    update contactsToUpdate;
}
```

---

### 4. **Prevent Duplicate Vehicles Under the Same Contact**

```apex
trigger PreventDuplicateVehicles on Vehicle__c (before insert, before update) {
    Set<String> uniqueKeys = new Set<String>();
    for (Vehicle__c veh : Trigger.new) {
        String uniqueKey = veh.Contact__c + '-' + veh.Make__c + '-' + veh.Model__c + '-' + veh.VIN__c;
        if (uniqueKeys.contains(uniqueKey)) veh.addError('Duplicate vehicle under this contact is not allowed.');
        uniqueKeys.add(uniqueKey);
    }
}
```

---

### 5. **Accept or Reject Service on Service Record**

1. **Add `Status__c` Picklist** on Service object (values: "Accepted," "Rejected").
2. **Add Validation Rule** to prevent status changes if "Accepted" or "Rejected" is already set.

---

### 6. **Reports and Dashboard**

1. **Create Reports**:
   - Monthly Sales, Sales by Area, Sales by Salesperson.

2. **Dashboard Components**:
   - Visualize sales trends using charts based on created reports.

---

This concise code and structure follow best practices, ensuring efficient data management and trigger performance. Each trigger is lean, focused, and handles necessary business logic without overlap.
