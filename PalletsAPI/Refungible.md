# Refungible

Refungible pallet is used to manage refungible tokens and their collections. It offers functionality to:
- Create/destroy collections
- Create/destroy items (mint/burn tokens)
- Manage collection/token properties
- Transfer tokens
- Repartition RFT
- Manage collection admins list

Pallet users are divided into several roles:
- **Collection Owner** - collection creator. Only whitelist admins are allowed to create collections (see [Whitelist]).
- **Collection Admin** - privileged user of a collection. Added by collection owner. Only whitelist admins are allowed to be collection admins (see [Whitelist]). **Collection owner also is a collection admin** so for example "Collection admins are allowed to use this call" equals "Collection admins and collection owner are...".
- **Investor** - originally is whitelist investor (see [Whitelist]). Collection admins and collection owner are also have investor privilege meaning are able to transfer and set allowance.

Each pallet call has required permission level which is satisfied by some role. Pallet provides the following functions:
|Call|Roles|
|---|---|
|initCollection|Whitelist Admin|
|destroyCollection|Collection Owner|
|setCollectionProperty|Collection Admin|
|deleteCollectionProperty|Collection Admin|
|setCollectionProperties|Collection Admin|
|deleteCollectionProperties|Collection Admin|
|setPropertyPermission|Collection Owner|
|createItem|Collection Admin|
|createMultipleItems|Collection Admin|
|setTokenProperty|Collection Admin|
|setTokenProperties|Collection Admin|
|deleteTokenProperty|Collection Admin|
|deleteTokenProperties|Collection Admin|
|transfer|Investor|
|transferFrom|Investor|
|setAllowance|Investor|
|burn|Collection Admin|
|burnFrom|Collection Admin|
|repartition|Collection Admin|
|toggleAdmin|Collection Owner|


> ⚠️ Notes for `Example` sections: 
> - We are using some testing helper objects and functions such as `accounts`, `state`, `stringToUnicodeArr`...
They are not a part of API and used here to explicitly prepare required state without extra details.
> - Also there are `txOption` const that is passed to `signAndSend` method. It equals to `{nonce: -1}` and used to make the provider be aware about transactions already in the transaction pool (more [here][more on txOptions reason]).


---
## 💠 ***initCollection***

### **Description**

Creates a new collection. Creator must be whitelisted admin and will be charged for collection creation (amount will be specified when public testnet is up).

### **Declaration**

`initCollection(data: CreateCollectionData)`

- **data: [CreateCollectionData]** - some collection parameters

### **Usage**

**Prerequisites**
- Sender is a whitelist admin

**Event**: [CollectionCreated]

**Errors**: [initCollection errors]

### **Example**
```typescript
// just creating an account with some tokens to pay the creation and transaction fees
const [alice] = await accounts.deriveNewAccounts([1000n]);

// only whitelisted admins are allowed to create collections
// explicitly making alice an admin for example
await state.setWhitelistAdmins([alice.address], accounts.root);

const collectionData = {
    // for this pallet must be 'refungible'
    mode: 'refungible',
    name: stringToUnicodeArr('Collection name'),
    description: stringToUnicodeArr('Collection description'),
    // Yes, token prefix could be passed as a string unlike name and description
    // See CreateCollectionData docs
    tokenPrefix: 'ECT',
    properties: [
        {
            key: 'Key', 
            value: 'Value'
        }
    ],
    propertyPermissions: [
        // let mutate the 'Key' property specified above
        {
            key: 'Key',
            permission: {
                mutable: true
            }
        },
        // let add this property later
        {
            key: 'NewKey',
            permission: {
                mutable: true
            }
        }
    ]
}

// init a collection
await api.tx.refungible
    .initCollection(collectionData)
    .signAndSend(alice, txOptions);
```
---
## 💠 **destroyCollection**

### **Description**

Destroys a collection. Collection have to not contain any tokens (they all must be previously burned) to destroy a collection. Only collection creator (owner) is able to destroy it.

### **Declaration**

`destroyCollection(collectionId: CollectionId)`

- **collectionId: [CollectionId]**

### **Usage**

**Prerequisites**
- Sender is a collection owner
- There is a collection with passed `collectionId`
- There are no tokens in this collection

**Event**: [CollectionDestroyed]

**Errors**: [destroyCollection errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });

// destroy a collection
await api.tx.refungible
    .destroyCollection(collectionId)
    .signAndSend(alice, txOptions);
```

---
## 💠 **setCollectionProperty**

### **Description**

Set or update a collection property value. Property must be mutable to be able to do this. Property permission (mutable or not) is set during collection initialization or through `setPropertyPermission`. Only collection admins or collection owner are able to set collection properties.

### **Declaration**

`setCollectionProperty(collectionId: CollectionId, property: Property)`

- **collectionId: [CollectionId]**
- **property: [Property]**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- Property under passed property key is mutable

**Event**: [CollectionPropertySet]

**Errors**: [setCollectionProperty errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
// mutable_properties field here let us add a property under 'Key' key later
// under the hood it is passed to `propertyPermissions` from `CreateCollectionData` (see InitCollection)
const collectionId = await state.createCollection(alice, { mode: 'refungible', mutableProperties: ['Key'] });

// prepare the a property
const property = {key: 'Key', value: "Value"};
// set collection property
await api.tx.refungible
    .setCollectionProperty(collectionId, property)
    .signAndSend(alice, txOptions);
```

---
## 💠 **deleteСollectionProperty**

### **Description**

Deletes a collection property value. Only collection admins or collection owner are able to delete collection properties.

### **Declaration**

`deleteCollectionProperty(collectionId: CollectionId, propertyKey: PropertyKey)`

- **collectionId: [CollectionId]**
- **propertyKey: [PropertyKey]**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- There is a property with passed property key

**Event**: [CollectionPropertyDeleted]

**Errors**: [deleteCollectionProperty errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// prepare a propery
const property = {key: 'Key', value: 'Value'};
// and init a collection with this property
const collectionId = await state.createCollection(alice, { mode: 'refungible', properties: [property] });

// delete the property
await api.tx.refungible
    .deleteCollectionProperty(collectionId, property.key)
    .signAndSend(alice, txOptions);
```

---
## 💠 **setCollectionProperties**

### **Description**

Batch call for `setCollectionProperty`.

### **Declaration**

`setCollectionProperties(collectionId: CollectionId, properties: Property[])`

- **collectionId: [CollectionId]**
- **properties: [Property] []**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- Properties under passed property keys are mutable

**Event**: [CollectionPropertySet]

**Errors**: [setCollectionProperty errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
// mutable_properties explained in setCollectionProperty
const collectionId = await state.createCollection(alice, { mode: 'refungible', mutableProperties: ['Key1', 'Key2'] });

// prepare the a properties array
const properties = [{key: 'Key1', value: 'Value'}, {key: 'Key2', value: 'Value'}];
// set collection properties
await api.tx.refungible
    .setCollectionProperties(collectionId, properties)
    .signAndSend(alice, txOptions);
```

---
## 💠 **deleteCollectionProperties**

### **Description**

Batch call for `deleteCollectionProperty`.

### **Declaration**

`deleteCollectionProperty(collectionId: CollectionId, propertyKeys: PropertyKey[])`

- **collectionId: [CollectionId]**
- **propertyKeys: [PropertyKey] []**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- There are properties under passed property keys

**Event**: [CollectionPropertyDeleted]

**Errors**: [deleteCollectionProperty errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// prepare a properties array
const properties = [{key: 'Key1', value: 'Value'}, {key: 'Key2', value: 'Value'}];
// and init a collection with these properties
const collectionId = await state.createCollection(alice, { mode: 'refungible', properties });

// delete collection properties
await api.tx.refungible
    .deleteCollectionProperties(collectionId, properties.map(p => p.key))
    .signAndSend(alice, txOptions);
```

---
## 💠 **setPropertyPermission**

### **Description**

Update a collection property permission which is made it mutable or not. Actually it is possible only to restrict property permission to unmutable as unmutable properties and their permissions can not be changed. Only collection owner can do this.

### **Declaration**

`setPropertyPermission(collectionId: CollectionId, propertyPermission: PropertyKeyPermission)`

- **collectionId: [CollectionId]**
- **propertyPermission: [PropertyKeyPermission]** - key and permission for property under this key

### **Usage**

**Prerequisites**
- Sender is a collection owner
- There is a collection with passed `collectionId`
- Property under passed property key is mutable

**Event**: [PropertyPermissionSet]

**Errors**: [setPropertyPermission errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
// mutable_properties explained in setCollectionProperty
const collectionId = await state.createCollection(alice, { mode: 'refungible', mutableProperties: ['Key'] });

// prepare a PropertyKeyPermission
const propertyPermission = {
    key: 'Key',
    permission: {
        mutable: false
    }
};
// set property permission
await api.tx.refungible
    .setPropertyPermission(collectionId, propertyPermission)
    .signAndSend(alice, txOptions);
```

---
## 💠 **createItem**

### **Description**

Mint a token in some collection. Initial user balances are passed and their sum comprises a total supply. Only collection admins and a collection owner are allowed to mint tokens.

### **Declaration**

`createItem(collectionId: CollectionId, data: CreateItemData)`

- **collectionId: [CollectionId]**
- **data: [CreateItemData]**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- Properties under passed property keys are mutable

**Event**: [ItemCreated]

**Errors**: [createItem errors]

### **Example**
```typescript
// get accounts
const [alice, bob] = await accounts.deriveNewAccounts([1000n, 0n]);
// make a collection creator a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// make bob a whitelisted investor
// as only collection admins, collection owner and whitelisted investors are
// allowed to have security tokens
await state.setWhitelistInvestors([bob.address], alice);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });

const data = {
    balances: [
        // for the first account
        [alice.address, 100n], 
        // for the second account
        [bob.address, 50n]
        // for the third account could be...
    ],
    properties: [{key: 'Key', value: 'Value'}]
};
// create an item
await api.tx.refungible
    .createItem(collectionId, data)
    .signAndSend(alice, txOptions);
```

---
## 💠 **createMultipleItems**

### **Description**

Batch call for `createItem`

### **Declaration**

`createItems(collectionId: CollectionId, usersBalances: [AccountId, TokenBalance][][], tokensProperties: Property[][])`

- **collectionId: [CollectionId]**
- **data: [CreateItemData] []**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- Properties under passed property keys are mutable

**Event**: [ItemCreated]

**Errors**: [createItem errors]

### **Example**
```typescript
// get accounts
const [alice, bob] = await accounts.deriveNewAccounts([1000n, 0n]);
// make a collection creator a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// make bob a whitelisted investor
// as only collection admins, collection owner and whitelisted investors are
// allowed to have security tokens
await state.setWhitelistInvestors([bob.address], alice);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });

// tokens data
const data = [
    // for the first token 
    {
        balances: [[alice.address, 100n], [bob.address, 50n]],
        properties: [{key: 'Key1', value: 'Value1'}]
    },
    // for the second token
    {
        balances: [[alice.address, 30n], [bob.address, 20n]],
        properties: [{key: 'Key2', value: 'Value2'}]
    }
    // for the third token could be...
];

// create items
await api.tx.refungible
    .createMultipleItems(collectionId, data)
    .signAndSend(alice, txOptions);
```

---
## 💠 **setTokenProperty**

### **Description**

Set or update a token property value. Property must be mutable to be able to do this. Property permission (mutable or not) is set during collection initialization or through `setPropertyPermission`. Property permissions is common for collection and its tokens meaning that there is only one permission under one for no matter it is a collection or a token property key. Only collection admins or collection owner are able to set token properties.

### **Declaration**

`setTokenProperty(collectionId: CollectionId, tokenId: TokenId, property: Property)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **property: [Property]**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Properties under passed property keys are mutable

**Event**: [TokenPropertySet]

**Errors**: [setTokenProperty errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
// mutableProperties explained in setCollectionProperty
// property permissions are common for a collection and its tokens so passed here
const collectionId = await state.createCollection(alice, { mode: 'refungible', mutableProperties: ['Key'] });
// create a token an get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[alice.address, 100n]]);

// prepare a Property
const property = {key: 'Key', value: 'Value'};
// set token property
await api.tx.refungible
    .setTokenProperty(collectionId, tokenId, property)
    .signAndSend(alice, txOptions);
```

---
## 💠 **setTokenProperties**

### **Description**

Batch call for `setTokenProperty`.

### **Declaration**

`setTokenProperties(collectionId: CollectionId, tokenId: TokenId, properties: Property[])`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **properties: [Property][]**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Properties under passed property keys are mutable

**Event**: [TokenPropertySet]

**Errors**: [setTokenProperty errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
// mutableProperties explained in setCollectionProperty
// property permissions are common for a collection and its tokens so passed here
const collectionId = await state.createCollection(alice, { mode: 'refungible', mutableProperties: ['Key1', 'Key2'] });
// create a token an get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[alice.address, 100n]]);

// prepare a Properties
const properties = [{key: 'Key1', value: 'Value1'}, {key: 'Key2', value: 'Value2'}];
// set token properties
await api.tx.refungible
    .setTokenProperties(collectionId, tokenId, properties)
    .signAndSend(alice, txOptions);
```

---
## 💠 **deleteTokenProperty**

### **Description**

Deletes a token property value. Only collection admins or collection owner are able to delete token properties.

### **Declaration**

`deleteTokenProperty(collectionId: CollectionId, tokenId: TokenId, propertyKey: PropertyKey)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **propertyKey: [PropertyKey]**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Properties under passed property keys are mutable

**Event**: [TokenPropertyDeleted]

**Errors**: [deleteTokenProperty errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
// mutableProperties explained in setCollectionProperty
// property permissions are common for a collection and its tokens so passed here
const collectionId = await state.createCollection(alice, { mode: 'refungible', mutableProperties: ['Key'] });
// prepare a propety
const property = {key: 'Key', value: 'Value'};
// create a token with some property an get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[alice.address, 100n]], [property]);

// delete token property
await api.tx.refungible
    .deleteTokenProperty(collectionId, tokenId, property.key)
    .signAndSend(alice, txOptions);
```

---
## 💠 **deleteTokenproperties**

### **Description**

Batch call for `deleteTokenProperty`

### **Declaration**

`deleteTokenProperty(collectionId: CollectionId, tokenId: TokenId, propertyKey: PropertyKey)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **propertyKey: [PropertyKey]**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Properties under passed property keys are mutable

**Event**: [TokenPropertyDeleted]

**Errors**: [deleteTokenProperty errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make the account a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
// mutableProperties explained in setCollectionProperty
// property permissions are common for a collection and its tokens so passed here
const collectionId = await state.createCollection(alice, { mode: 'refungible', mutableProperties: ['Key1', 'Key2'] });
// prepare a propety
const properties = [{key: 'Key1', value: 'Value1'}, {key: 'Key2', value: 'Value2'}];
// create a token with some property an get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[alice.address, 100n]], properties);
// delete token properties
await api.tx.refungible
    .deleteTokenProperties(collectionId, tokenId, properties.map(p => p.key))
    .signAndSend(alice, txOptions);
```

---
## 💠 **transfer**

### **Description**

Transfer RFT token from sender account to target account. As only collection admins, collection owner and active whitelisted investors are allowed to hold security tokens transfer receiver must also be one of them.

### **Declaration**

`transfer(collectionId: CollectionId, tokenId: TokenId, to: AccountId, amount: TokenBalance)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **to: [AccountId]**
- **amount: [TokenBalance]**

### **Usage**

**Prerequisites**
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Sender have enough tokens
- Receiver is an _investor_

**Event**: [Transfer]

**Errors**: [transfer errors]

### **Example**
```typescript
// get accounts
const [alice, bob] = await accounts.deriveNewAccounts([1000n, 0n]);
// make alice a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// make future token holders are whitelisted investors
await state.setWhitelistInvestors([bob.address], alice);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });
// create a token and get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[alice.address, 100n]]);

// transfer
await api.tx.refungible
    .transfer(collectionId, tokenId, bob.address, 70n)
    .signAndSend(alice, txOptions);
```

---
## 💠 **transferFrom**

### **Description**

Augmented tranfer call allowing to transfer from account which gived sender an allowance.

### **Declaration**

`transferFrom(collectionId: CollectionId, tokenId: TokenId, from: AccountId, to: AccountId, amount: TokenBalance)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **from: [AccountId]** - who gived an allowance
- **to: [AccountId]**
- **amount: [TokenBalance]**

### **Usage**

**Prerequisites**
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Sender have sufficient allowance
- Receiver is an _investor_

**Event**: [Transfer]

**Errors**: [transferFrom errors]

### **Example**
```typescript
// get accounts
const [alice, bob, eve] = await accounts.deriveNewAccounts([1000n, 1000n, 0n]);
// make alice a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// make future token holders are whitelisted investors
await state.setWhitelistInvestors([bob.address, eve.address], alice);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });
// create a token and get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[alice.address, 100n]]);

// set allowance to call transferFrom
await api.tx.refungible
    .setAllowance(collectionId, tokenId, bob.address, 70n)
    .signAndSend(alice, txOptions);

// transfer
await api.tx.refungible
    .transferFrom(collectionId, tokenId, alice.address, eve.address, 70n)
    .signAndSend(bob, txOptions);
```

---
## 💠 **setAllowance**

### **Description**

This call let a token holder give some other account an allowance which give them a permission to transfer holder's tokens using `transferFrom`. Account taking an allowance should also be allowed token holder meaning the account must to be one of collection owner, collection admin or active whitelisted admin.

### **Declaration**

`setAllowance(collectionId: CollectionId, tokenId: TokenId, spender: AccountId, amount: TokenBalance)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **spender: [AccountId]** - who get allowance
- **amount: [TokenBalance]**

### **Usage**

**Prerequisites**
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Sender have enough tokens
- Allowance receiver is an _investor_

**Event**: [Approved]

**Errors**: [setAllowance errors]

### **Example**
```typescript
// get accounts
const [alice, bob] = await accounts.deriveNewAccounts([1000n, 0n]);
// make alice a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// make future token holders are whitelisted investors
await state.setWhitelistInvestors([bob.address], alice);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });
// create a token and get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[alice.address, 100n]]);

// set allowance
await api.tx.refungible
    .setAllowance(collectionId, tokenId, bob.address, 70n)
    .signAndSend(alice, txOptions);
```

---
## 💠 **burn**

### **Description**

Burn/destroy some tokens removing them from total supply. Only collection owner and collection admins are able to burn tokens.

### **Declaration**

`burn(collectionId: CollectionId, tokenId: TokenId, amount: TokenBalance)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **amount: [TokenBalance]**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Sender have enough tokens

**Event**: [ItemDestroyed]

**Errors**: [burn errors]

### **Example**
```typescript
// get an account
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make alice a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });
// create a token and get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[alice.address, 100n]]);

// burn
await api.tx.refungible
    .burn(collectionId, tokenId, 100n)
    .signAndSend(alice, txOptions);
```

---
## 💠 **burnFrom**

### **Description**

Burn approved tokens. Only collection owner and collection admins are able to burn tokens.

### **Declaration**

`burnFrom(collectionId: CollectionId, tokenId: TokenId, from: AccountId, amount: TokenBalance)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **from: [AccountId]**
- **amount: [TokenBalance]**

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Sender have sufficient allowance

**Event**: [ItemDestroyed]

**Errors**: [burnFrom errors]

### **Example**
```typescript
// get accounts
const [alice, bob] = await accounts.deriveNewAccounts([1000n, 1000n]);
// make alice a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// make future token holders are whitelisted investors
await state.setWhitelistInvestors([bob.address], alice);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });
// create a token and get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[bob.address, 100n]]);

// set an allowance
const approveUnSub: CompleteCallback = await api.tx.refungible
    .setAllowance(collectionId, tokenId, alice.address, 70n)
    .signAndSend(bob, txOptions);
// burn
await api.tx.refungible
    .burnFrom(collectionId, tokenId, bob.address, 70n)
    .signAndSend(alice, txOptions);
```

---
## 💠 **repartition**

### **Description**

Changes the token pieces count. Only collection owner or collection admins holding all of the token pieces are able to do a repartition.

### **Declaration**

`repartition(collectionId: CollectionId, tokenId: TokenId, amount: TokenBalance)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **amount: [TokenBalance]** - new pieces amount

### **Usage**

**Prerequisites**
- Sender is a collection admin
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- Sender have all token pieces

**Event**: [ItemCreated] if new pieces amount is bigger than the old one and [ItemDestroyed] if less

**Errors**: [repartition errors]

### **Example**
```typescript
// get accounts
const [alice] = await accounts.deriveNewAccounts([1000n]);
// make alice a whitelisted admin
await state.setWhitelistAdmins([alice.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });
// create a token and get a tokenId from ItemCreated event
const tokenId = await state.createItem(alice, collectionId, [[alice.address, 100n]]);

// repartition
await api.tx.refungible
    .repartition(collectionId, tokenId, 50n)
    .signAndSend(alice, txOptions);
```

---
## 💠 **toggle_admin**

### **Description**

Toggle some account's participation in collection's admin list. Only collection owner is able to change collection admin list.

### **Declaration**

`toggleAdmin(collectionId: CollectionId, tokenId: TokenId, user: AccountId, admin: bool)`

- **collectionId: [CollectionId]**
- **tokenId: [TokenId]**
- **admin: [AccountId]** - true -> make an admin, false -> remove from admin list

### **Usage**

**Prerequisites**
- Sender is a collection owner
- There is a collection with passed `collectionId`
- There is a token with passed `TokenId`
- New admin is a whitelist admin

**Event**: [AdminToggled]

**Errors**: [toggleAdmin errors]

### **Example**
```typescript
// get accounts
const [alice, bob] = await accounts.deriveNewAccounts([1000n, 0n]);
// make alice a whitelisted admin to create collection
// also make bob a whitelisted admin as only whitelisted admins could be collection admins
await state.setWhitelistAdmins([alice.address, bob.address], accounts.root);
// create a collection an get a collectionId from CollectionCreated event
const collectionId = await state.createCollection(alice, { mode: 'refungible' });

// add an admin
await api.tx.refungible
    .toggleAdmin(collectionId, bob.address, true)
    .signAndSend(alice, txOptions);
```

<!-- Links -->
<!-- Other pages -->
[Whitelist]: Whitelist.md
[more on txOptions reason]: https://polkadot.js.org/docs/api/cookbook/tx#how-do-i-take-the-pending-tx-pool-into-account-in-my-nonce

<!-- Types -->
[AccountId]: ./TypesReference.md#accountid
[CollectionId]: ./TypesReference.md#collectionid
[CreateCollectionData]: ./TypesReference.md#createcollectiondata
[CreateItemData]: ./TypesReference.md#createitemdata
[Property]: ./TypesReference.md#property
[PropertyKey]: ./TypesReference.md#propertykey
[PropertyKeyPermission]: ./TypesReference.md#propertykeypermission
[TokenId]: ./TypesReference.md#tokenid
[TokenBalance]: ./TypesReference.md#tokenbalance

<!-- Events -->
[CollectionCreated]: http://www.reddit.com
[CollectionDestroyed]: http://www.reddit.com
[ItemCreated]: http://www.reddit.com
[ItemDestroyed]: http://www.reddit.com
[CollectionPropertySet]: http://www.reddit.com
[CollectionPropertyDeleted]: http://www.reddit.com
[TokenPropertySet]: http://www.reddit.com
[TokenPropertyDeleted]: http://www.reddit.com
[PropertyPermissionSet]: http://www.reddit.com
[Transfer]: http://www.reddit.com
[Approved]: http://www.reddit.com
[AdminToggled]: http://www.reddit.com


<!-- Errors -->
[initCollection errors]: http://www.reddit.com
[destroyCollection errors]: http://www.reddit.com
[setCollectionProperty errors]: http://www.reddit.com
[deleteCollectionProperty errors]: http://www.reddit.com
[createItem errors]: http://www.reddit.com
[setTokenProperty errors]: http://www.reddit.com
[deleteTokenProperty errors]: http://www.reddit.com
[setPropertyPermission errors]: http://www.reddit.com
[transfer errors]: http://www.reddit.com
[transferFrom errors]: http://www.reddit.com
[setAllowance errors]: http://www.reddit.com
[burn errors]: http://www.reddit.com
[burnFrom errors]: http://www.reddit.com
[repartition errors]: http://www.reddit.com
[toggleAdmin errors]: http://www.reddit.com
