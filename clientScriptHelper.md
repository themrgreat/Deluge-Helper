### Documentation

- [Zoho CRM Client Script - API](https://www.zohocrm.dev/explore/client-script/clientapi/Page#getField)


---


# Take items from Deal Record (Sub-Form) and Create Element Record with Name & Category :

```javascript
console.log(".................START.................");

const stage = ZDK.Page.getField("Stage").getValue();

if (stage !== "Closed Won") {
    ZDK.Client.showAlert(`Deal must be "Closed Won" to proceed.`);
    // console.log ("It's not Closed Won");
    return;
}

const items = ZDK.Page.getSubform("Items").getValues();
// console.log(items);

let elementNameArray = [];
let elementCategoryArray = [];

if (items.length !== 0) {

items.forEach(item => {
    const prodName = item.Product_Name_1;
    const prodCategory = item.Product_Category;

    //     console.log(item);
    //     console.log(`prodName : ${prodName} ,  prodCategory : ${prodCategory}`);

    elementNameArray.push(prodName);
    elementCategoryArray.push(prodCategory);
});
} else {
    ZDK.Client.showAlert("There is no items in Sub-Form named items");
        // console.log("There is no items in Sub-Form named items");
        return;
}

if (elementNameArray.length > 0 && elementCategoryArray.length > 0) {
    const element = new ZDK.Apps.CRM.Elements.Models.Elements();

    element.Name = elementNameArray.join(", ");
    element.Element_Category = elementCategoryArray.join(", ");

    const res = ZDK.Apps.CRM.Elements.create(element);
    // console.log(res[0]._status);
    ZDK.Client.showMessage("Element record created successfully.");
} else {
    ZDK.Client.showAlert("Something went wrong!");
    return;
}

console.log(".................END.................");
```

---

# Set value (Amount Field) in Quotes Create Page :

```javascript
Note : Create quotes in deal record.

console.log(".................START.................");

const dealName = ZDK.Page.getField("Deal_Name").getValue();
// console.log(dealName.id);

var parentId = $Page.parent_record_id;
console.log(parentId);
console.log($Crm.user);
console.log($Page.record_id);

const dealData = ZDK.Apps.CRM.Deals.fetchById(dealName.id);
// console.log(dealData._Amount);

const amount = dealData._Amount;

if (amount !== null || amount !== undefined) {
    ZDK.Page.getField("Amount").setValue(amount);
} else {
    console.log("Something went wrong!")
}

console.log(".................END.................");
```

---

# Auto Fill Pickup List in ClientScript (Part - 1) :

```javascript
const service = ZDK.Page.getField("Practices_and_Services").getValue();
// console.log(service);

if (service) {
    const product = ZDK.Apps.CRM.Products.fetchById(service.id);
    console.log(product);
    const practice = product._Practice_Area;
    const subPractice = product._Sub_Practice_Area;
    console.log(practice);
    console.log(subPractice);
    // if (practice) {
        ZDK.Page.getField("Practice_Area").setValue(practice);
    // }
    // if (subPractice) {
        ZDK.Page.getField("Sub_Practice_Area").setValue(subPractice);
    // }
}
```
---

# Auto Fill Fields and Lookup in ClientScript (Part - 2) :

```javascript
console.log(".................START.................");
 
// DEAL DATA SECTION
 
const dealLookup = ZDK.Page.getField("Deal_Name").getValue();
 
if (!dealLookup || !dealLookup.id) {
    console.log("No Deal linked to this Case.");
} else {
 
    try {
 
        const dealData = ZDK.Apps.CRM.Deals.fetchById(dealLookup.id);
 
        const productSegment = dealData._Product_Segment;
        const opportunityType = dealData._Opportunity_Type;
        const panelType = dealData._Panel_Type;
        const quoteData = dealData._Quote_Data;
 
        if (quoteData && quoteData.length > 0) {
 
            const productLookup = quoteData[0].Product;
 
            if (productLookup && productLookup.id) {
 
                const productData = ZDK.Apps.CRM.Products.fetchById(productLookup.id);
                const rating = productData._RATING;
 
                ZDK.Page.getField("Product_Name").setValue(productLookup);
                ZDK.Page.getField("Product_Segment").setValue(productSegment);
                ZDK.Page.getField("Opportunity_Type").setValue(opportunityType);
                ZDK.Page.getField("Panel_Type").setValue(panelType);
                ZDK.Page.getField("KVA").setValue(rating);
            }
        }
 
    } catch (error) {
        console.log("Deal Section Error: " + error.message);
    }
}


// CONTACT DATA SECTION
 
const contactLookup = ZDK.Page.getField("Contact_Name").getValue();
 
if (!contactLookup || !contactLookup.id) {
    console.log("No Contact linked to this Case.");
} else {
 
    try {
 
        const contactData = ZDK.Apps.CRM.Contacts.fetchById(contactLookup.id);
 
        const email = contactData._Email;
        const mobile = contactData._Mobile;
        const firstName = contactData._First_Name || "";
        const lastName = contactData._Last_Name || "";
 
        const fullName = (firstName + " " + lastName).trim();
 
        // Set Case Fields
        ZDK.Page.getField("Customer_Name").setValue(fullName);
        ZDK.Page.getField("Customer_E_mail").setValue(email);
        ZDK.Page.getField("Customer_Mobile_No").setValue(mobile);
 
        // Auto Subject (customize as needed)
        ZDK.Page.getField("Subject").setValue("Complaint - " + fullName);
 
        // Set Current Date
        const today = new Date();
        // .toLocaleString('en-US', { month: 'short', day: 'numeric', year: 'numeric' });
        // console.log(today.toLocaleDateString('en-GB'));
        ZDK.Page.getField("Date_of_Complaint").setValue(today.toLocaleDateString('en-GB'));
 
        const month = today.toLocaleString('en-US', { month: 'short' }); 
        // console.log(month); // Feb
        ZDK.Page.getField("Month").setValue(month);
 
    } catch (error) {
        console.log("Contact Section Error: " + error.message);
    }
}
 
ZDK.Client.showMessage("Fields auto-filled successfully.");
 
console.log(".................END.................");

```


---

# Find Duplicates in Leads & Contacts Module using Clientscript (Part - 1) :

```javascript
const leadMobile = ZDK.Page.getField("Mobile").getValue();
const leadOwner = ZDK.Page.getField("Owner").getValue();

const leads = ZDK.Apps.CRM.Leads.fetch();
const contacts = ZDK.Apps.CRM.Contacts.fetch();

////////////////////////////////////////////////////////////

let isLeadDuplicate = false;
let isContactDuplicate = false;

leads.forEach(lead => {
    const mobile = lead._Mobile;
    if (leadMobile && mobile && leadMobile === mobile) {
        isLeadDuplicate = true;
    }
});

contacts.forEach(contact => {
    const mobile = contact._Mobile;
    if (leadMobile && mobile && leadMobile === mobile) {
        isContactDuplicate = true;
    }
});

///////////////////////////////////////////////////////////

const isLeadDuplicate = leads.some(lead => {
    const mobile = lead._Mobile;
    return leadMobile && mobile && leadMobile === mobile;
});

const isContactDuplicate = contacts.some(contact => {
    const mobile = contact._Mobile;
    return leadMobile && mobile && leadMobile === mobile;
});

let duplicateName = "";

if (isLeadDuplicate && isContactDuplicate) {
    duplicateName = "Lead & Contact Record";
    ZDK.Client.showAlert(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
} else if (isLeadDuplicate) {
    duplicateName = "Lead Record";
    ZDK.Client.showAlert(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
} else if (isContactDuplicate) {
    duplicateName = "Contact Record";
    ZDK.Client.showAlert(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
}


```


---

# Find Duplicates in Leads & Contacts Module using Clientscript (Part - 2) :

```javascript
const leadMobile = ZDK.Page.getField("Mobile").getValue();
const leadMobileField = ZDK.Page.getField("Mobile");

const leadOwner = ZDK.Page.getField("Owner").getValue();

let isLeadDuplicate = false;
let isContactDuplicate = false;

try {
    const leads = await zrc.get('/crm/v8/Leads/search', {
        params: { phone: leadMobile, fields: 'Owner,Mobile' }
    });
    // console.log(leads.data.data);
    if (leads.data && leads.data.data.length > 0) {
        isLeadDuplicate = true;
    }
} catch (error) {
    console.log("Error fetching leads: " + error.message);
}

try {
    const contacts = await zrc.get('/crm/v8/Contacts/search', {
        params: { phone: leadMobile, fields: 'Owner,Mobile' }
    });
    if (contacts.data && contacts.data.data.length > 0) {
        isContactDuplicate = true;
    }
} catch (error) {
    console.log("Error fetching contacts: " + error.message);
}

let duplicateName = "";

if (isLeadDuplicate && isContactDuplicate) {
    duplicateName = "Lead & Contact Record";
    leadMobileField.showError(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
} else if (isLeadDuplicate) {
    duplicateName = "Lead Record";
    leadMobileField.showError(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
} else if (isContactDuplicate) {
    duplicateName = "Contact Record";
    leadMobileField.showError(`This mobile number is already in ${duplicateName}, Owner : ${leadOwner.name}.`);
}
```

---

# Search using COQL :

```javascript
// Define the query and parameters
const parameters = {
    "select_query": "select Last_Name, First_Name from Contacts where Last_Name is not null limit 3"
};

// Replace "connection_name" with the name you defined in the Connections setup
const response = ZDK.Apps.CRM.Connections.invoke(
    "connection_name",
    "https://www.zohoapis.com/crm/v6/coql",
    "POST",
    2,
    parameters,
    {}
);

console.log(response);
```
---