# Buyer Group

Buyer Groups can be made based on:
1. Same Person Of Contact.
2. Same PAN number.
3. Change in Invoice.

## 01. Group Limit
The group limit is the sum of the two highest limits of entities in the group.

###  Why are Group Limits needed?
Group Limits are introduced so that all the entities in a group are limited to a fixed amount and can't order above that available amount unless it is approved by the users with designation/role as **Admin / Buyer Operations / Bussiness Team**.

This is done to avoid *Concentrated Risk*.

<details><summary>Click For Code</summary>
<p>

```js
function _calculateGroupLimit(groupTotalLimit, combinedData) {
    return groupTotalLimit - combinedData.totalReceivables;
```
```js
function _calculateGroupTotalLimit(PANWiseData) {
      let groupTotalLimit = 0;
      if (PANWiseData.length > 1) {
        const topPANs = _.orderBy(
          PANWiseData,
          [e => e.creditProfile.totalLimit],
          ["desc"]
        ).slice(0, 2);
        groupTotalLimit = sumArray(topPANs.map(e => e.creditProfile.totalLimit));
      } else if (PANWiseData.length === 1) {
        // not possible to have a length other than this
        groupTotalLimit = PANWiseData[0].creditProfile.totalLimit;
      }
      return groupTotalLimit;
    }
```  

</p>
</details>   
<br>


## 02. Group Payment Category

The group payment categories are '*ADVANCED*' and '*DELIVERED*'.

A new group has a default payment category as *'DELIVERY + 1'*.

<details><summary>Click For Code</summary>
<p>

```js
async _createNewGroup(buyerEntityObj, PAN, creditProfileSection, session) {
    const buyerGroupObj = {
      entityIds: [buyerEntityObj.vendorId],
      personIds: [],
      relationships: [],
      creditProfile: {
        PANLimits: {
          [PAN]: {
            ...creditProfileSection,
          },
        },
        tradeReferences: [],
        groupPaymentCategory: GROUP_PAYMENT_CATEGORY.DELIVERY_PLUS_1,
      },
      blacklisted: false,
    };
    return Object.create(this.buyerGroup)
      .setUser(this.user)
      .createGroup(buyerGroupObj, { session });
}
```

</p>
</details>   
<br>

If in a group an entity places order againts advance then the Group Payment Category is changed to ADVANCED for all entities.

<details><summary>Click For Code</summary>
<p>

```js
  async recomputeGroupPaymentCategory(
    newGroupPaymentCategory,
    existingGroupPaymentCategory
  ) {
    if (
      existingGroupPaymentCategory === GROUP_PAYMENT_CATEGORY.ADVANCE ||
      newGroupPaymentCategory === GROUP_PAYMENT_CATEGORY.ADVANCE
    )
      return GROUP_PAYMENT_CATEGORY.ADVANCE;
    else if (
      newGroupPaymentCategory === GROUP_PAYMENT_CATEGORY.DELIVERY_PLUS_1 &&
      existingGroupPaymentCategory !== GROUP_PAYMENT_CATEGORY.DELIVERY_PLUS_1
    )
      return existingGroupPaymentCategory;
    else if (
      existingGroupPaymentCategory === GROUP_PAYMENT_CATEGORY.DELIVERY_PLUS_1 &&
      newGroupPaymentCategory !== GROUP_PAYMENT_CATEGORY.DELIVERY_PLUS_1
    )
      return newGroupPaymentCategory;
    else {
      return newGroupPaymentCategory;
    }
  }
```
</p>
</details>   
<br>

--------(still remaining)---------
   
<br>

## 03. GST Slabs
Entities are alloted a slab accoring to their yearly turn-over.   
<br>

The slabs are as follows:

| Sr. No.       | Slabs 
| ------------- | ------------- |
| SLAB0 | 0-40L 
| SLAB1 | 40L - 1.5Cr 
| SLAB2 | 1.5Cr - 5Cr 
| SLAB3 | 5Cr - 25Cr 
| SLAB4 | 25Cr - 100Cr 
| SLAB5 | 100Cr - 500Cr 
| SLAB6 | More than 500Cr 

<br>

## 04. How is Slab Limit alloted?
Slab limit is alloted by using a formula:
- [(Median*75%/12) *25%], 
where 'Median' is the median of the slab range.

For eg. :

If the Slab alloted is 5-25Cr, then, 

**Median == 15**

then, 

__Slab Limit == [(15Cr*75%/12) *25%]__

* But we Round off for Order size.

<br>

Following is the table for Slab Limit:

| FY22 Slabs   | Slab Limit  |
|:------------:|:-----------:|
| 0-40l        | 0           |
| 40-1.5cr     | 0           |
| 1.5-5cr      | 8L          |
| 5-25cr       | 30L         |
| 25-100cr     | 50L         |
| 100-500cr    | 50L         |
| 500cr+       | 50L         |

* After 25-100Cr slab we allot the DCL(Default Credit Limit) as the Slab Limit.

   
<br>

## 05. Total Limit

Total Limit is the limit upto which an entity can place order/s.

Total limit depends on the Insaurance Limit, Credit Limit or the Slab Limit   

Total limit will be:
- Insurance Limit --> when Credit Limit is empty.
- Credit Limit --> when Insuarnce Limit is empty.
- The minimum limit --> when both Insurance Limit and Credit limit are available (i.e. usually credit limit).
- Slab Limit --> when both Insurance Limit and Slab Limit are empty.   
<br>

Below Code snippet explains the implemented logic:
<details><summary>Click For Code</summary>
<p>

```js
const entityPAN = entityWiseData.filter(e => e.vendorId === entityId)[0].PAN;
const entityPANData = PANWiseData.filter(e => e.PAN === entityPAN)[0];
```
```js
const PANTotalLimit = calculatePANTotalLimit(entityPANData.creditProfile);
```
```js
function _calculatePANLimit({ creditProfile, totalReceivables }) {
      const { insuranceLimit, creditLimit, creditLimitEnabled } = creditProfile;
      const limit =
        creditLimit === 0 && insuranceLimit === null && creditLimitEnabled === false
          ? DCLLimit
          : calculatePANTotalLimit(creditProfile);
      return limit - totalReceivables;
    }

    function _calculatePANOutstanding({ totalReceivables }) {
      return totalReceivables;
    }
  };
```
```js
function calculatePANTotalLimit(creditProfile) {
  const {
    insuranceLimit,
    creditLimit,
    creditLimitEnabled,
    fy20Slab,
    fy21Slab,
    fy22Slab,
  } = creditProfile;
  const preferredSlab = getPreferredSlabValue(fy22Slab, fy21Slab, fy20Slab);

  if (creditLimit !== 0 && creditLimitEnabled)
    if (insuranceLimit !== null) return Math.min(creditLimit, insuranceLimit);
    else return creditLimit;
  else {
    if (insuranceLimit === null) return slabToTotalLimit[fy22Slab] ?? 0;
    else return insuranceLimit;
  }
}

```
</p>
</details>   
<br>

## 06. Available Limit
Available Limit is calculated by the formula:
  * Available Limit == min[(Group Limit - Group Outstanding), X],
  
  where, X is found by:

      * If entity has an Insuarance limit:
      then, X = [Insuarance limit - (Entity Wise Outstanding)]
  OR

      * If entity does not have an Insuarance Limit:
      then, X = [DLC - (Entity Wise Outstanding)] 

Following code snipet explains the logic:
<details><summary>Click For Code</summary>
<p>

```js
const PANWiseData = Object.entries(
      buyerGroup.getObject().creditProfile.PANLimits
    ).map(([PAN, creditProfile]) => {
      const buyerEntityData = entityWiseData.filter(e => e.PAN === PAN);
      const PANDueData = buyerEntityData.reduce(
        (a, b) => ({
          overdueAmount: a.overdueAmount + b.overdueAmount,
          totalReceivables: a.totalReceivables + b.totalReceivables,
          overdueCases: a.overdueCases + b.overdueCases,
        }),
        { overdueAmount: 0, totalReceivables: 0, overdueCases: 0 }
      );
      return {
        PAN,
        creditProfile,
        ...PANDueData,
      };
    });

const combinedData = entityWiseData.reduce(
      (a, b) => ({
        overdueAmount: a.overdueAmount + b.overdueAmount,
        totalReceivables: a.totalReceivables + b.totalReceivables,
        overdueCases: a.overdueCases + b.overdueCases,
      }),
      { overdueAmount: 0, totalReceivables: 0, overdueCases: 0 }
    );

const entityPANData = PANWiseData.filter(e => e.PAN === entityPAN)[0];
```
```js
const PANLimit = _calculatePANLimit(entityPANData);

const groupTotalLimit = _calculateGroupTotalLimit(PANWiseData);

const groupLimit = _calculateGroupLimit(groupTotalLimit, combinedData);

function _calculatePANLimit({ creditProfile, totalReceivables }) {
      const { insuranceLimit, creditLimit, creditLimitEnabled } = creditProfile;
      const limit =
        creditLimit === 0 && insuranceLimit === null && creditLimitEnabled === false
          ? DCLLimit
          : calculatePANTotalLimit(creditProfile);
      return limit - totalReceivables;
    }

```
</p>
</details>   
<br>

Thus available limit will be computed by using following code:
```js
const PANAvailableLimit = Math.min(groupLimit, PANLimit);
```   
<br>

## 07. Credit Limit

Credit Limit is the limit which is approved by