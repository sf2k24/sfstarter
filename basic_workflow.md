### 1. Display Total Payments Made and Count of Rejected Services on Account

This functionality involves aggregating payments and rejected service records per account.

- **Setup**:
  1. **Create Custom Fields** on the **Account** object:
     - **Total_Payments__c** (Currency): Stores the total payments made for all accepted services.
     - **Rejected_Services_Count__c** (Number): Tracks the count of rejected services associated with the account.

- **Apex Trigger**:
  - Write an Apex trigger on the **Service** object to update these fields on **Account** whenever a **Service** record is created, updated, or deleted.
  - Use an aggregate SOQL query to calculate total payments and rejected services for each account.

- **Code Example**:
  ```apex
  trigger UpdateAccountStats on Service__c (after insert, after update, after delete, after undelete) {
      // Step 1: Gather Account IDs affected by the trigger
      Set<Id> accountIds = new Set<Id>();
      for (Service__c service : Trigger.isDelete ? Trigger.old : Trigger.new) {
          if (service.Account__c != null) {
              accountIds.add(service.Account__c);
          }
      }

      // Step 2: Aggregate Payments and Rejected Service Count
      List<Account> accountsToUpdate = new List<Account>();
      if (!accountIds.isEmpty()) {
          // Query to sum payments and count rejected services for each account
          AggregateResult[] results = [
              SELECT Account__c, SUM(Payment_Amount__c) totalPayments,
                     COUNT(Id) rejectedCount
              FROM Service__c
              WHERE Account__c IN :accountIds AND Status__c = 'Rejected'
              GROUP BY Account__c
          ];
          
          // Step 3: Update Account fields based on aggregated results
          for (AggregateResult ar : results) {
              Account acc = new Account(Id = (Id) ar.get('Account__c'));
              acc.Total_Payments__c = (Decimal) ar.get('totalPayments');
              acc.Rejected_Services_Count__c = (Integer) ar.get('rejectedCount');
              accountsToUpdate.add(acc);
          }
      }
      update accountsToUpdate;
  }
  ```

### 2. Display Total Number of Vehicles Serviced and Rejected Services on Account

This functionality requires counting all vehicles that have received service and rejected service records associated with each **Account**.

- **Setup**:
  - Create a **Vehicles_Serviced_Count__c** field on the **Account** object.

- **Apex Trigger Update**:
  - Extend the trigger on **Service** to add logic to count serviced vehicles.

- **Additional Code Logic**:
  ```apex
  AggregateResult[] vehicleCounts = [
      SELECT Account__c, COUNT_DISTINCT(Vehicle__c) vehicleCount
      FROM Service__c
      WHERE Account__c IN :accountIds AND Status__c = 'Accepted'
      GROUP BY Account__c
  ];

  for (AggregateResult vr : vehicleCounts) {
      Account acc = new Account(Id = (Id) vr.get('Account__c'));
      acc.Vehicles_Serviced_Count__c = (Integer) vr.get('vehicleCount');
      accountsToUpdate.add(acc);
  }
  update accountsToUpdate;
  ```

---

### 3. Prevent Duplicate Contacts under an Account

This functionality requires a trigger on **Contact** to prevent duplicate entries based on criteria like **Email** within the same account.

- **Setup**:
  - Write a `before insert` and `before update` trigger on **Contact** to validate uniqueness.

- **Code Example**:
  ```apex
  trigger PreventDuplicateContacts on Contact (before insert, before update) {
      Map<Id, Set<String>> accountEmails = new Map<Id, Set<String>>();

      for (Contact c : Trigger.new) {
          if (c.AccountId != null && c.Email != null) {
              if (!accountEmails.containsKey(c.AccountId)) {
                  accountEmails.put(c.AccountId, new Set<String>());
              }
              accountEmails.get(c.AccountId).add(c.Email);
          }
      }

      List<Contact> existingContacts = [SELECT AccountId, Email FROM Contact 
                                        WHERE AccountId IN :accountEmails.keySet()];

      for (Contact ec : existingContacts) {
          if (accountEmails.containsKey(ec.AccountId) && accountEmails.get(ec.AccountId).contains(ec.Email)) {
              Trigger.newMap.get(ec.Id).addError('Duplicate contact with the same email under this account.');
          }
      }
  }
  ```

---

### 4. Show Total Amount Spent on Services for Each Contact

To track total spending by each contact, calculate the sum of **Payment Amount** for services linked to each contact.

- **Setup**:
  - Create **Total_Amount_Spent__c** field on the **Contact** object.
  - Update this field whenever a **Service** record is added or updated.

- **Trigger Code**:
  ```apex
  trigger UpdateContactSpend on Service__c (after insert, after update, after delete) {
      Map<Id, Decimal> contactSpending = new Map<Id, Decimal>();

      for (Service__c service : [SELECT Contact__c, SUM(Payment_Amount__c) totalSpent
                                 FROM Service__c WHERE Contact__c != null GROUP BY Contact__c]) {
          contactSpending.put(service.Contact__c, (Decimal) service.get('totalSpent'));
      }

      List<Contact> contactsToUpdate = new List<Contact>();
      for (Id contactId : contactSpending.keySet()) {
          Contact con = new Contact(Id = contactId);
          con.Total_Amount_Spent__c = contactSpending.get(contactId);
          contactsToUpdate.add(con);
      }
      update contactsToUpdate;
  }
  ```

---

### 5. Prevent Duplicate Vehicles under a Contact

This functionality prevents duplicate vehicles under each contact by checking the **VIN** field.

- **Trigger**:
  - Write a `before insert` and `before update` trigger on **Vehicle** to check for duplicates.

- **Code Example**:
  ```apex
  trigger PreventDuplicateVehicles on Vehicle__c (before insert, before update) {
      Set<String> contactVehicleVINs = new Set<String>();

      for (Vehicle__c v : Trigger.new) {
          if (v.Contact__c != null && v.VIN__c != null) {
              String key = v.Contact__c + ':' + v.VIN__c;
              if (contactVehicleVINs.contains(key)) {
                  v.addError('Duplicate vehicle detected with the same VIN for this contact.');
              }
              contactVehicleVINs.add(key);
          }
      }
  }
  ```

---

### 6. Accept or Reject Service Requests

To enable accepting or rejecting a service request, add a **Status__c** field (Picklist) to the **Service** object, with options like *Accepted* and *Rejected*.

- **Trigger or Handler Logic**:
  - Ensure that the **Status** field updates related information accordingly. For instance, update the **Rejected_Services_Count__c** on **Account** whenever a service is rejected.

---

### 7. Reports and Dashboards for Sales Metrics

Salesforce's reporting and dashboard features handle sales metrics such as total sales per month, area, and salesperson.

- **Setup**:
  - **Create Custom Report Types**:
    - Build reports based on custom fields for insights into monthly sales, geographic sales distribution, and performance by salesperson.
  - **Dashboard Creation**:
    - Design dashboards to visualize the aggregated report data.

This setup provides a comprehensive implementation of the functionalities for the **Vehicle Service Station App** using **Apex**, ensuring both data accuracy and ease of use across various user roles.



  ```apex
trigger UpdateAccountStats on Service__c (after insert, after update, after delete, after undelete) {
    // Step 1: Gather Account IDs affected by the trigger
    Set<Id> accountIds = new Set<Id>();

    if (Trigger.isDelete) {
        // When records are deleted, use Trigger.old to get account IDs
        for (Service__c service : Trigger.old) {
            if (service.Account__c != null) {
                accountIds.add(service.Account__c);
            }
        }
    } else {
        // For insert, update, and undelete, use Trigger.new to get account IDs
        for (Service__c service : Trigger.new) {
            if (service.Account__c != null) {
                accountIds.add(service.Account__c);
            }
        }
    }

    if (!accountIds.isEmpty()) {
        // Step 2: Aggregate Total Payments and Count of Rejected Services
        // Total Payments: Sum of Payment Amount for each Account where Status is "Accepted"
        AggregateResult[] paymentResults = [
            SELECT Account__c, SUM(Payment_Amount__c) totalPayments
            FROM Service__c
            WHERE Account__c IN :accountIds AND Status__c = 'Accepted'
            GROUP BY Account__c
        ];

        // Count of Rejected Services
        AggregateResult[] rejectedServiceResults = [
            SELECT Account__c, COUNT(Id) rejectedCount
            FROM Service__c
            WHERE Account__c IN :accountIds AND Status__c = 'Rejected'
            GROUP BY Account__c
        ];

        // Count of Unique Vehicles Serviced
        AggregateResult[] vehicleCounts = [
            SELECT Account__c, COUNT_DISTINCT(Vehicle__c) vehicleCount
            FROM Service__c
            WHERE Account__c IN :accountIds AND Status__c = 'Accepted'
            GROUP BY Account__c
        ];

        // Step 3: Map results to Account objects
        Map<Id, Account> accountsToUpdate = new Map<Id, Account>();

        // Update Total Payments
        for (AggregateResult ar : paymentResults) {
            Id accountId = (Id) ar.get('Account__c');
            Decimal totalPayments = (Decimal) ar.get('totalPayments');

            if (!accountsToUpdate.containsKey(accountId)) {
                accountsToUpdate.put(accountId, new Account(Id = accountId));
            }
            accountsToUpdate.get(accountId).Total_Payments__c = totalPayments;
        }

        // Update Rejected Services Count
        for (AggregateResult ar : rejectedServiceResults) {
            Id accountId = (Id) ar.get('Account__c');
            Integer rejectedCount = (Integer) ar.get('rejectedCount');

            if (!accountsToUpdate.containsKey(accountId)) {
                accountsToUpdate.put(accountId, new Account(Id = accountId));
            }
            accountsToUpdate.get(accountId).Rejected_Services_Count__c = rejectedCount;
        }

        // Update Unique Vehicles Serviced Count
        for (AggregateResult ar : vehicleCounts) {
            Id accountId = (Id) ar.get('Account__c');
            Integer vehicleCount = (Integer) ar.get('vehicleCount');

            if (!accountsToUpdate.containsKey(accountId)) {
                accountsToUpdate.put(accountId, new Account(Id = accountId));
            }
            accountsToUpdate.get(accountId).Vehicles_Serviced_Count__c = vehicleCount;
        }

        // Step 4: Update Accounts with new Total Payments, Rejected Services Count, and Vehicles Serviced Count
        update accountsToUpdate.values();
    }
}
```

