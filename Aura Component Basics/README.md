# **[Refactor components and communicate with events](https://trailhead.salesforce.com/content/learn/modules/lex_dev_lc_basics/lex_dev_lc_basics_events)**

## Refactor the input form for camping list items into its own component and communicate with component events.

- Replace the HTML form in the campingList component with a new campingListForm component that calls the clickCreateItem JavaScript controller action when clicked
- The campingList component listens for a c: addItemEvent event and executes the action handleAddItem in the JavaScript controller
- The handleAddItem method saves the record to the database and adds the record to the items value provider
- The addItemEvent event is of type component and has a Camping_Item\_\_c type attribute named item
- The campingListForm registers an addItem event of type c: addItemEvent
- The campingListFormController JavaScript controller calls the helper's createItem method if the form is valid
- The campingListForm Helper JavaScript helper creates an addItem event with the item to be added, fires the event, then resets the newItem value provider with a blank sObjectType of type Camping_Item\_\_c

### **Camping List**

_CampingList.cmp_

```java
<aura:component controller="CampingListController">

    <!-- Attribute Definition -->
    <aura:attribute name="items" type="Camping_Item__c[]"/>

    <!-- Handlers -->
    <aura:handler name="init" action="{!c.doInit}" value="{!this}" />
    <aura:handler name="addItem" event="c:addItemEvent" action="{!c.handleAddItem}"/>

    <!-- Page Header -->
    <div class="slds-page-header" role="banner" >
        <div class="slds-grid">
            <div class="slds-col" >
                <h1 class="slds-text-heading--medium">Camping List</h1>
            </div>
        </div>
    </div>

    <!-- New Camping Form -->
    <c:campingListForm />

    <!-- Camping List Items -->
    <aura:iteration items="{!v.items}" var="itm">
          <c:campingListItem item="{!itm}"/><br/>
    </aura:iteration>

</aura:component>
```

_CampingListController.js_

```javascript
({
  // load camping list items on initiation
  doInit: function (component, event, helper) {
    var action = component.get("c.getItems");
    action.setCallback(this, function (response) {
      var state = response.getState();
      if (component.isValid() && state === "SUCCESS") {
        component.set("v.items", response.getReturnValue());
        console.log("doInit: " + response.getReturnValue());
      }
    });
    $A.enqueueAction(action);
  },

  // handle adding items to camping list
  handleAddItem: function (component, event, helper) {
    var item = event.getParam("item");

    var action = component.get("c.saveItem");
    action.setParams({
      item: item,
    });

    action.setCallback(this, function (response) {
      var state = response.getState();
      if (component.isValid() && state === "SUCCESS") {
        var items = component.get("v.items");
        items.push(response.getReturnValue());
        component.set("v.items", items);
      }
    });
    $A.enqueueAction(action);
  },
});
```

_CampingListHelper.js_

```javascript
({
  createItem: function (component, campaign) {
    this.saveItem(component, campaign, function (response) {
      var state = response.getState();
      if (component.isValid() && state === "SUCCESS") {
        var campaigns = component.get("v.items");
        var retcamp = response.getReturnValue();
        campaigns.push(retcamp);
        component.set("v.items", campaigns);
      }
    });
  },

  saveItem: function (component, campaign, callback) {
    var action = component.get("c.saveItem");
    action.setParams({
      campaign: campaign,
    });

    if (callback) {
      action.setCallback(this, callback);
    }
    $A.enqueueAction(action);
  },
});
```

### **Camping List Form**

_CampingListForm.cmp_

```java
<aura:component controller="CampingListController" >

    <!-- Attribute Definition -->
    <aura:attribute name="newItem"  type="Camping_Item__c" default="{ 'sobjectType': 'Camping_Item__c',
                                                                    'Name':'',
                                                                    'Quantity__c': 0,
                                                                    'Price__c': 0,
                                                                    'Packed__c': false}"  />

    <!-- Register Handlers -->
    <aura:registerEvent name="addItem" type="c:addItemEvent"/>


    <!-- New Camping Form-->
    <div class="slds-col slds-col--padded slds-p-top--large" >
        <fieldset class="slds-box slds-theme--default slds-container--small">
            <legend id="newexpenseform" class="slds-text-heading--small" >
                Add Camping List
            </legend>

            <form class="slds-form--stacked">

                <!-- Name -->
                <div class="slds-form-element slds-is-required" >
                    <div class="slds-form-element__control" >
                        <lightning:input aura:id="itemform"
                        label="Name"
                        name="itemname"
                        value="{!v.newItem.Name}"
                        required="true"/>
                    </div>
                </div>

                <!-- Quantity -->
                <div class="slds-form-element__label" >
                    <div class="slds-form-element__control" >
                    <lightning:input type="number"
                    aura:id="itemform"
                    label="Quantity"
                    name="quantity"
                    value="{!v.newItem.Quantity__c}"
                    min="1"
                    required="true"/>
                    </div>
                </div>

                <!-- Price -->
                <div class="slds-form-element slds-is-required" >
                    <div class="slds-form-element__control" >
                    <lightning:input type="number"
                    aura:id="itemform"
                    label="Price"
                    name="price"
                    value="{!v.newItem.Price__c}"
                    formatter="currency"
                    step="0.01"/>
                    </div>
                </div>

                <!-- Packed -->
                <div class="slds-form-element" >
                    <lightning:input type="checkbox"
                    aura:id="itemform"
                    label="Packed?"
                    name="packed"
                    checked="{!v.newItem.Packed__c}"/>
                </div>

                <!-- Button Create Expense -->
                <div class="slds-form-element">
                    <lightning:button label="Create Camping Item"
                        variant="brand"
                        onclick="{!c.clickCreateItem}"/>
                </div>

            </form>
        </fieldset>
    </div>

</aura:component>
```

_CampingListFormController.js_

```javascript
({
  clickCreateItem: function (component, event, helper) {
    var validCampign = true;

    // Name cannot be blank
    var nameField = component.find("name");
    var campaignname = nameField.get("v.value");
    if ($A.util.isEmpty(campaignname)) {
      validCampign = false;
      nameField.set("v.errors", [{ message: "Camping name can't be blank." }]);
    } else {
      nameField.set("v.errors", null);
    }

    // Quantity cannot be blank.
    var qty = component.find("quantity");
    var quantity = qty.get("v.value");
    if ($A.util.isEmpty(quantity)) {
      validCampign = false;
      qty.set("v.errors", [{ message: "Quantity can't be blank." }]);
    } else {
      qty.set("v.errors", null);
    }

    // Price cannot be blank.
    var priceField = component.find("price");
    var price = priceField.get("v.value");
    if ($A.util.isEmpty(price)) {
      validCampign = false;
      priceField.set("v.errors", [{ message: "Price can't be blank." }]);
    } else {
      priceField.set("v.errors", null);
    }

    // No validation errors
    if (validCampign) {
      var newCampaign = component.get("v.newItem");
      helper.createItem(component, newCampaign);
    }
  },
});
```

_CampingListFormHelper.js_

```javascript
({
  createItem: function (component, item) {
    var createEvent = component.getEvent("addItem");
    createEvent.setParams({ item: item });
    createEvent.fire();
    component.set("v.newItem", {
      sobjectType: "Camping_Item__c",
      Name: "",
      Quantity__c: 0,
      Price__c: 0,
      Packed__c: false,
    });
  },
});
```

### **Event**

_addItemEvent.evt_

```java
<aura:event type="COMPONENT">
    <aura:attribute name="item" type="Camping_Item__c"/>
</aura:event>
```
