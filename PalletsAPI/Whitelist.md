# Whitelist

Whitelist pallet is used to manage security tokens whitelist. It is used in other pallets to control security assets transfers and possesion.

There are following roles in the pallet:
- **RolesRoot** - superuser able to assign Admins. Roles root hardcoded in genesis and could not be changed later.
- **Admin** - control Managers and Investors.
- **Manager** - less privileged role that only add investors and change their status.
- **Investor** - user, holder of security asset. Investor could have `Active` status or not. Not active investors simply disabled for some reason and can not use their privilege, but may be made `Active` later by whitelist admin or manager.
  
Pallet provides the following functions:

|Call|Roles|
|---|---|
|addAdmin|RolesRoot|
|removeAdmin|RolesRoot|
|addManager|Admin|
|removeManager|Admin|
|addInvestors|Admin or Manager|
|setInvestorStatus|Admin or Manager|
|changeInvestorAddress|Admin|
|changeMyAddress|Investor|


> ⚠️ Notes for `Example` sections: 
> - We are using some testing helper objects and functions such as `accounts`, `state`, `stringToUnicodeArr`...
They are not a part of API and used here to explicitly prepare required state without extra details.
> - Also there are `txOption` const that is passed to `signAndSend` method. It equals to `{nonce: -1}` and used to make the provider be aware about transactions already in the transaction pool (more [here][more on txOptions reason]).


---
## 💠 ***addAdmin***

### **Description**

Make passed account a whitelist admin. Only RolesRoot is able to add or remove admins.

### **Declaration**

`addAdmin(newAdmin: AccountId)`

- **`newAdmin`: [AccountId]** - admin to be added

### **Usage**
**Prerequisites**
- Sender is a RolesRoot
- `newAdmin` is not an admin

**Event**: [AddAdmin]

**Errors**: [addAdmin errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([0n]);

// accounts.root is a RolesRoot of the whitelist pallet
// add alice to admin list
await api.tx.whitelist
    .addAdmin(alice.address)
    .signAndSend(accounts.root, txOptions);
```
---
## 💠 ***removeAdmin***

### **Description**

Remove existing whitelist admin. Only RolesRoot is able to add or remove admins.

### **Declaration**

`removeAdmin(admin: AccountId)`

- **`admin`: [AccountId]** - admin to be removed

### **Usage**
**Prerequisites**
- Sender is a RolesRoot
- `admin` is an admin

**Event**: [RemoveAdmin]

**Errors**: [removeAdmin errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([0n]);
// make alice a whitelist admin to remove later
await state.setWhitelistAdmins([alice.address], accounts.root);

// accounts.root is a RolesRoot of the whitelist pallet
// remove alice from admin list
await api.tx.whitelist
    .removeAdmin(alice.address)
    .signAndSend(accounts.root, txOptions);
```

---
## 💠 ***addManager***

### **Description**

Make passed account a whitelist manager. Only admins are able to add or remove managers.

### **Declaration**

`addManager(newManager: AccountId)`

- **newManager: [AccountId]** - manager to be added

### **Usage**

**Prerequisites**
- Sender is an admin
- `newManager` is not a manager

**Event**: [AddManager]

**Errors**: [addManager errors]

### **Example**
```typescript
// get accounts
const [alice, bob] = await accounts.deriveNewAccounts([1000n, 0n]);
// make alice an admin
await state.setWhitelistAdmins([alice.address], accounts.root);

// add bob to manager list
await api.tx.whitelist
    .addManager(bob.address)
    .signAndSend(alice, txOptions);
```

---
## 💠 ***removeManager***

### **Description**

Remove existing whitelist manager. Only admins are able to add or remove managers.

### **Declaration**

`removeManager(manager: AccountId)`

- **manager: [AccountId]** - manager to be removed

### **Usage**

**Prerequisites**
- Sender is an admin
- `manager` is a manager

**Event**: [RemoveManager]

**Errors**: [removeManager errors]

### **Example**
```typescript
// get accounts
const [alice, bob] = await accounts.deriveNewAccounts([1000n, 0n]);
// make alice an admin
await state.setWhitelistAdmins([alice.address], accounts.root);

// make bob a manager to remove later
await api.tx.whitelist
    .addManager(bob.address)
    .signAndSend(alice, txOptions);

// remove bob from managers list
await api.tx.whitelist
    .removeManager(bob.address)
    .signAndSend(alice, txOptions);
```

---
## 💠 ***addInvestors***

### **Description**

Make passed accounts active whitelist investors. Admins and managers are allowed to add investors.

### **Declaration**

`addInvestors(newInvestors: [InvestorKey, Investor][])`

- **newInvestors: [[InvestorKey], [Investor]][]** - investors data to be added

### **Usage**

**Prerequisites**
- Sender is either an admin or a manager
- There are no [InvestorKey] duplicates in a `newInvestors`
- There are no [Investor] duplicates in a `newInvestors`
- There are no [Investor] that already investors  in a `newInvestors`

**Event**: [AddInvestor]

**Errors**: [addInvestors errors]

### **Example**
```typescript
// get accounts
const [alice, bob, eve] = await accounts.deriveNewAccounts([1000n, 0n, 0n,]);
// make alice an admin
await state.setWhitelistAdmins([alice.address], accounts.root);

// prepate investors data
const investors: [string, {account: string, isActive: boolean}][] = [
    [
        'B'.repeat(32),
        {
            account: bob.address,
            isActive: true
        }
    ],
    [
        'E'.repeat(32),
        {
            account: eve.address,
            isActive: true
        }
    ]
]

// add bob and eve to investors list
await api.tx.whitelist
    .addInvestors(investors)
    .signAndSend(alice, txOptions);
```

---
## 💠 ***setInvestorStatus***

### **Description**

Makes investor active or not. Admins and managers are allowed to change investor status.

### **Declaration**

`setInvestorStatus(investorAccount: AccountId, isActive: bool)`

- **investorAccount: [AccountId]** - account of investor whose status is going to be changed
- **isActive: bool** - true -> `Active`, false -> not

### **Usage**

**Prerequisites**
- Sender is either an admin or a manager
- `investorAccount` is an investor
- `isActive` is opposite to current investor status

**Event**: [InvestorStatusSet]

**Errors**: [setInvestorStatus errors]

### **Example**
```typescript
// get accounts
const [alice, bob] = await accounts.deriveNewAccounts([1000n, 0n]);
// make alice an admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// make bob an active investor
await state.setWhitelistInvestors([bob.address], alice);

// change bob status to not active
await api.tx.whitelist
    .setInvestorStatus(bob.address, false)
    .signAndSend(alice, txOptions);
```

---
## 💠 ***changeInvestorAddress***

### **Description**

Replace targer investor account address. Admins and an investor themself are able to change investor account address. This call is purposed for admins.

### **Declaration**

`changeInvestorAddress(investorAccount: AccountId, newAccount: AccountId)`

- **investorAccount: [AccountId]** - current investor account
- **newAccount: [AccountId]** - new investor account

### **Usage**

**Prerequisites**
- Sender is an admin
- `investorAccount` is an investor
- `newAccount` is not an investor

**Event**: [InvestorAccountChanged]

**Errors**: [changeInvestorAddress errors]

### **Example**
```typescript
// get accounts
const [alice, bob, eve] = await accounts.deriveNewAccounts([1000n, 0n, 0n]);
// make alice an admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// make bob an investor
await state.setWhitelistInvestors([bob.address], alice);

// change bob address
await api.tx.whitelist
    .changeInvestorAddress(bob.address, eve.address)
    .signAndSend(alice, txOptions);
```

---
## 💠 ***changeMyAddress***

### **Description**

If the sender is active investor - replace their account address with the new one. This call is purposed for investors.

### **Declaration**

`changeMyAddress(newAccount: AccountId)`

- **newAccount: [AccountId]** - new investor account

### **Usage**

**Prerequisites**
- Sender is an investor
- `newAccount` is not an investor

**Event**: [InvestorAccountChanged]

**Errors**: [changeMyAddress errors]

### **Example**
```typescript
// get accounts
const [alice, bob, newBob] = await accounts.deriveNewAccounts([1000n, 1000n, 0n]);
// make alice an admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// make bob an investor
await state.setWhitelistInvestors([bob.address], alice);

// bob is changing his address to a new one
await api.tx.whitelist
    .changeMyAddress(newBob.address)
    .signAndSend(bob, txOptions);
```

<!-- Links -->
<!-- Other pages -->
[more on txOptions reason]: https://polkadot.js.org/docs/api/cookbook/tx#how-do-i-take-the-pending-tx-pool-into-account-in-my-nonce

<!-- Types -->
[AccountId]: ./TypesReference.md#accountid
[Investor]: ./TypesReference.md#investor
[InvestorKey]: ./TypesReference.md#investorkey

<!-- Events -->
[AddAdmin]: http://www.reddit.com
[RemoveAdmin]: http://www.reddit.com
[AddManager]: http://www.reddit.com
[RemoveManager]: http://www.reddit.com
[AddInvestor]: http://www.reddit.com
[InvestorStatusSet]: http://www.reddit.com
[InvestorAccountChanged]: http://www.reddit.com

<!-- Errors -->
[addAdmin errors]: http://www.reddit.com
[removeAdmin errors]: http://www.reddit.com
[addManager errors]: http://www.reddit.com
[removeManager errors]: http://www.reddit.com
[addInvestors errors]: http://www.reddit.com
[setInvestorStatus errors]: http://www.reddit.com
[changeInvestorAddress errors]: http://www.reddit.com
[changeMyAddress errors]: http://www.reddit.com
