# Buyer Module

## 01. Send Grade Prices Message

Sends a template message with Grade Prices of the products to the Buyers selected according to the condition.

Send Grade Prices Message module has:

1. Targeted Location
    * Message will only be sent to buyers in selected location/s.
    * If the option "All" is selected, the message will be sent to all the buyers.   
<br>  

2. Targeted Grade Group
    * Message will be sent to the buyers who have subscribed to the selected grade group.
    * Multiple Grade Groups can also be selected.
    * If the option "All" is selected, the message will be sent to all the buyers. 
<br>

3. Buyers
    * Message will be sent to the particular selected buyer.
    * 3 characters need to typed to get the list of required buyers.
    * Multiple buyers can be selected.
<br>

4. Ignored Buyers
    * Selected buyers will not be sent the grade price message even if they come under the selected location.
    * 3 characters need to tyed to get the list of required buyers.
    * Multiple buyers can be selected.
<br>

After Submitting the message will be sent to the required buyers.    
<br>


## 02. Send Custom Message

Lets you send a customized price update message.

1. Message
    * Is a open text box. 
    * Message is written within the selected Template only.  
<br> 

2. Targeted location
    * Message will only be sent to buyers in selected location/s.
    * If the option "All" is selected, the message will be sent to all the buyers.   
<br>

3. Preview
    * The custom message typed in the 'Message' text area is displayed within a template in Preview.
    * It is a non editable text area.   
<br>
 
4. Targeted Grade Group
    * Message will be sent to the buyers who have subscribed to the selected grade group.
    * Multiple Grade Groups can also be selected.
    * If the option "All" is selected, the message will be sent to all the buyers. 
<br>

5. Buyers
    * Buyers can be selected from the list of buyers.
    * Multiple buyers can be selected.
    * Message will be sent to the selected buyer/s.   
<br>

6. Ignored Buyers
    * Buyers can be selected from the list of buyers.
    * Multiple buyers can be selected.
    * Message will not be sent to the selected buyer/s.   
<br>

7. Template Type
    * Required template can be selected from a drop down menu.
    * Selected template is displayed in the Preview text area.   
<br>

8. Targeted Volume Filter
    * ()

   
<br>

9. Targeted Volume Qty
    * ()   
<br>

10. Select Number
    * Message can be sent to BFO or BBO as per selected.   
<br>

## 03. Create Entity

1. GST Number
    * Takes input in 15 character GST format only.
    * Fetch button fetches all the reqired information from GST number.    
<br>

2. Company Name
    * Is fetched from GST number.   
<br>

3. Location 
    * Should match to GSTn state code.   
<br>

4. Grades
    * Grade/s can be selected according to buyer's requirement.   
<br>

5. Strong Grades
    * Grade/s can be selected according to buyer's requirement.   
<br>

6. Persons
    * Exsisting Person/s can be assigned to the new entity.
    * Person/s can be searched by entering first 3 characters.   
<br>

7. Payment Category
    * Payment Category can be selected as DELIVERED or ADVANCED accoringly from the dropdown.   
<br>

8. Registration date
    * It is fetched from the GST number.   
<br>

9. Business Type
    * It is a required field.
    * Business type options i.e 'Trader', 'Manufacturer' or 'Both' can be selected from a dropdown.   
<br>

After Submitting a new buyer entity is created.   
<br>

## 04. Edit Entity
'EDIT ENTITY' option is available once a buyer entity is selected.

1. Location
    * Can be changed but should match GSTn state code.   
<br>

2. Grades 
    * New grades can be added from the list.
    * Grades can be removed from the list.   
<br>

3. Strong Grdaes
    * Grades can be added or removed from the list.   
<br>

4. Payment Category
    * Can be changed from DELIVERED to ADVANCE and vice-verca.
    * Payment Category Remark is required after changing the payment category.   
<br>

5. Business Type
    * Can be changed from the dropdown options.   
<br>

After Submitting the buyer entity will be saved with the edited changes.   
<br>

## 05. Send Onboarding Message

A personalised template message welcoming the buyer will be sent to the selected buyer with a video link for buyer tutorial.

```js
export const sendBuyerOnboardingMessage = id => {
  return dispatch => {
    return API.post(`buyerEntity/${id}/sendOnboardingMessage`, {}).then(function () {
      dispatch({ type: "SEND_ONBOARDING_MESSAGE" });
    });
  };
};
```
```js
onSyncZoho = () => {
    this.props
      .zohoSync()
      .then(message => {
        alert(`${message ? message : "SUCCESS"}`);
      })
      .catch(() => {
        alert(`Error: Sync Failed. Error code`);
      });
  };
  onSendOnboardingMessage = id => {
    this.props
      .sendBuyerOnboardingMessage(id)
      .then(message => {
        alert(`${message ? message : "SUCCESS"}`);
      })
      .catch(() => {
        alert(`Error in buyer onboarding message`);
      });
  };
  ```
