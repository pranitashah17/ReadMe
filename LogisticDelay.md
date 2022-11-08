# Logistic Delay

## Case 1 : Updating the Order Status

* If the order status has any of these within Pending status: _[Pending(TA, Limit, FWD, Adv)]_ ,  
then the **Logistic Delay** score would **_not_** be calculated and will be displayed as blank in the orders table.
* As soon as the the status is updated to '_Pending(LA)_', Logistic Delay score will be calculated.   
Following are the conditions for _Pending_ status update:
    1. If the order status is updated to Pending (LA) before 1 pm and the Dispatch Date is the same day then the logistics delay score is 0.
    2. If the order status is updated to Pending (LA) after 1 pm and the Dispatch Date is the same day then the logistics delay score is -1.
    3. If the order status is updated to Pending (LA) before 1 pm and the Dispatch Date is the D+1 then the logistics delay score is -1.
    4. If the order status is updated to Pending (LA) after 1 pm and the Dispatch Date is the D+1 then the logistics delay score is -1.

* For split orders, consider the time when the status of an individual order is updated to either Pending (LA) or Enquiry Sent and calculate the Logistic Delay score accordingly.

* In cases where the order is split after sending an enquiry, the Logistic Delay score of the parent order should be updated in the child order.

* In the case of Parent-Child orders, the Logistics Delay score of the child order will always be updated as '0' as we are accounting for the changes in the parent order.   
<br>

## Case 2 : Updating the Dispatch Date

* If any of the users with the designation/role as '**Logistics**' updates the *Dispatch Date* of an order, the Logistics Delay score should increase by the number which is:  

      (New dispatch date - Existing Dispatch Date)     


* If any of the users with the designation/role of '**Buyer Operations / Admin / Super-Admin**' updates the *Dispatch Date* of an order, the Logistics Delay score should _**not**_ be changed.
* The above-mentioned cases should work even if the dispatch date is updated by editing an order or updating the dispatch date from Order module → Logistics functions → Update dispatch date  modal.

|                    | Change in LogisticsDispatch Delay Value |                          |          |
|-----------------------------------|--------------------------|--------------------------|----------|
| Current Dispatch Date             | Today                    | T+1                      | >T+1     |
| Increase by Non Logistic Profile  | No Change                | No Change                | No Change|
| Decrease by Non Logistics Profile | Not Allowed              | Reduce by Change Value   | No Change|
| Increase by Logistic Profile      | Increase by Change Value | Increase by Change Value | No change|
| Decrease by Logistic Profile      | Not Allowed              | Reduce by Change Value   | No Change|   
   
<br>

## Case 3 : Editing the order post confirmation

* In cases where the Buyer payment term are changed / Buyers are updated and the order status is again updated as '*Pending(x,y,z)*' then the Logistics Delay score calculation should **restart** following the above mentioned logics, i.e. when the status or the order is changed from Pending(Adv).   
<br>

## Case 4 : Common Points

* Min Value is -2, so even so convert All lower values (<-2) to -2.
* The Logistics delay score should stop updating when the order status is updated as "Vehicle Dispatched" and "Vehicle Delivered".

