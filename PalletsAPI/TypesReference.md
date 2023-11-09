# Types

## <a name="accountid"></a> :small_blue_diamond: **AccountId**

More on: [AccountId]

### **Usage**

**js type:** `number | number[] | string`

**examples**
```typescript
'5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY' // as a string
```

---
## <a name="collectionid"></a> :small_blue_diamond: **CollectionId**

More on: [CollectionId]

### **Usage**

**type:** any number

---

## <a name="createcollectiondata"></a> :small_blue_diamond: **CreateCollectionData**

More on: [CreateCollectionData]

### **Usage**

**js type**
```typescript
type bytes = string | number | number[];

type CreateCollectionData = {
    mode: 'refungible' | 'fungible' | 'nft',
    name: number[], // expects 2-bytes array
    description: number[], // expects 2-bytes array
    tokenPrefix: bytes,
    properties: ({key: bytes, value: bytes})[],
    propertyPermissions: ({
            key: PropertyKey, 
            permission: PropertyPermission
    })[]
};
```

**examples**
```typescript
function stringToUnicodeArr(str: string): number[] {
    return str.split('').map(c => c.charCodeAt(0));
}

const createCollectionData = {
    mode: 'refungible',
    name: stringToUnicodeArr('Collection name'),
    description: stringToUnicodeArr('Collection description'),
    tokenPrefix: 'ECT',
    properties: [
        {
            key: 'Key', 
            value: 'Value'
        }
    ],
    propertyPermissions: [
        {
            key: 'Key',
            permission: {
                mutable: true
            }
        }
    ]
};
```

---

## <a name="createItemData"></a> :small_blue_diamond: **CreateItemData**

More on: [CreateItemData]

### **Usage**

**js type**
```typescript
type CreateItemData = { 
    balances?: [AccountId, number][]; 
    properties?: Property[]
};
```

**examples**
```typescript
const createItemData = {
    balances: [
        [alice.address, 100n],
        [bob.address, 50n]
    ],
    properties: [
        {key: 'Key', value: 'Value'}
    ]
};
```

---
## <a name="investor"></a> :small_blue_diamond: **Investor**

More on: [Investor]

### **Usage**

**js type**
```typescript
type Investor = [
    InvestorKey, 
    {
        account: AccountId,
        isActive: boolean
    }
];
```

**examples**
```typescript
const investor = [
    'A'.repeat(32),
    {
        account: '',
        isActive: true
    }
];
```

---
## <a name="investorkey"></a> :small_blue_diamond: **InvestorKey**

More on: [InvestorKey]

### **Usage**

**js type:** 32 bytes as `number | number[] | string`

**examples**
```typescript
const strKey = 'A'.repeat(32);
```

---
## <a name="property"></a> :small_blue_diamond: **Property**

More on: [Property]

### **Usage**

**js type**
```typescript
type bytes = string | number | number[];

type Property = {
    key: bytes,
    value: bytes
};
```

**examples**
```typescript
const strProperty = {key: 'Key', value: 'Value'};

const combinedProperty = {key: 'Number', value: 1};
```

---
## <a name="propertykey"></a> :small_blue_diamond: **PropertyKey**

More on: [PropertyKey]

### **Usage**

**js type**
```typescript
type bytes = string | number | number[];

type PropertyKey = bytes;
```

**examples**
```typescript
const strKey = 'Key';

const numKey = 1;
```

---
## <a name="propertykeypermission"></a> :small_blue_diamond: **PropertyKeyPermission**

More on: [PropertyKeyPermission]

### **Usage**

**js type**
```typescript
type PropertyKeyPermission = {
    key: PropertyKey,
    permission: {
        mutable: boolean
    }
};
```

**examples**
```typescript
const propertyKeyPermission = {
    key: 'Key',
    permission: {
        mutable: false
    }
};
```

---
## <a name="tokenid"></a> :small_blue_diamond:**TokenId**

More on: [TokenId]

### **Usage**

**js type:** any number

---
## <a name="tokenbalance"></a> :small_blue_diamond: **TokenBalance**

More on: [TokenBalance]

### **Usage**

**js type:** any number

<!-- Links -->

<!-- Types -->
[AccountId]: http://www.reddit.com
[CollectionId]: http://www.reddit.com
[CollectionFlags]: http://www.reddit.com
[CreateCollectionData]: http://www.reddit.com
[CreateItemData]: http://www.reddit.com
[Investor]: http://www.reddit.com
[InvestorKey]: http://www.reddit.com
[Property]: http://www.reddit.com
[PropertyKey]: http://www.reddit.com
[PropertyKeyPermission]: http://www.reddit.com
[TokenId]: http://www.reddit.com
[TokenBalance]: http://www.reddit.com