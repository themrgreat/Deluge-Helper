# Single Lookup Update :

```javascript
 	sinContactID = "1136779000000569290";
 	accID = "1136779000000569096";

 	sinCon = zoho.crm.getRecordById("Contacts", sinContactID);
// 	info sinCon;

 	sinMap = Map();
 	sinMap.put("Lookup",sinContactID);
 //	sinMap.put("Lookup",{"id":sinContactID});
 	info sinMap;

 	updateSinCon = zoho.crm.updateRecord("Accounts", accID, sinMap);
 	info updateSinCon;
```

Json Format -- for Updating in Single Lookpup --


```json
{"Lookup":"1136779000000569290"}
```


---

# Create a lead with status "Fresh" :


```javascript
lead = Map();

lead.put("Company","Test Company");
lead.put("First_Name","Test");
lead.put("Last_Name","Testing");
lead.put("Email","test@email.com");
lead.put("Phone","1234567890");
lead.put("Lead_Status","Fresh");

res = zoho.crm.createRecord("Leads", lead);
info res;


```


---

# Reminder for Email and Popup :


```javascript
taskMap = Map();

taskMap.put("Subject","New task is created in lead");
taskMap.put("Description","15 minute automated reminder");
taskMap.put("Priority","High");
taskMap.put("Status","Not Started");
taskMap.put("Due_Date",zoho.currentdate);

taskMap.put("What_Id",leadId);
taskMap.put("$se_module","Leads");

rem = zoho.currenttime.addMinutes(15).toString("yyyy-MM-dd'T'HH:mm:ss+05:30");

taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=EMAILANDPOPUP;TRIGGER=DATE-TIME:" + rem});

info zoho.crm.createRecord("Tasks",taskMap);


```


---

# Create Quotes and Show in Deals :

```javascript
Note - Take Products(Related List) from Deal Record & create quotation with Quoted Items(Same from Related List from Products).

    dealId = "1136779000000569369";
    getProductData = zoho.crm.getRelatedRecords("Products", "Deals", dealId);
 
    info getProductData;
 
    productList = List();
    for each product in getProductData
    {
        productMap = Map();
        productMap.put("product", {"id": product.get("id")});
        productMap.put("quantity", 1);
//      productMap.put("list_price", product.get("Unit_Price").toDecimal());
        productList.add(productMap);
    }
 	info productList;
 
    quoteMap = Map();
    quoteMap.put("Subject", "Quote for Deal " + dealId);
    quoteMap.put("Deal_Name", dealId);
    quoteMap.put("Product_Details", productList);
 
    info quoteMap;
 
    res = zoho.crm.createRecord("Quotes", quoteMap);
    info res;
```

Json Format -- Create Quotes and Show in Deals --


```json
{
  "Subject": "Quote for Deal 1136779000000569369",
  "Deal_Name": "1136779000000569369",
  "Product_Details": [
    {
      "product": {
        "id": "1136779000000734026"
      },
      "quantity": 1
    },
    {
      "product": {
        "id": "1136779000000734021"
      },
      "quantity": 1
    },
    {
      "product": {
        "id": "1136779000000734016"
      },
      "quantity": 1
    },
    {
      "product": {
        "id": "1136779000000734011"
      },
      "quantity": 1
    }
  ]
}
```


---

# Add in Multi-Select Lookup in AccountsXContacts using invoke :


```javascript
accID = "1136779000000569096";
multContID1 = "1136779000000569290";
multContID2 = "1136779000000572469";
multContID3 = "1136779000000569287";

// response = invokeurl
// [
// 	url :"https://www.zohoapis.in/crm/v2.1/Accounts/" + accID
// 	type :GET
// 	connection:"getdata"
// ];
// info response;

contList = List();
cont1 = Map();
cont1.put("id",multContID1);
cont11 = Map();
cont11.put("Add_Contacts_1",cont1);
cont111 = Map();
cont111.put(cont11);
contList.add(cont111);
// info cont111;

cont2 = Map();
cont2.put("id",multContID2);
cont22 = Map();
cont22.put("Add_Contacts_1",cont2);
cont222 = Map();
cont222.put(cont22);
contList.add(cont222);
// info contList;

obj1Map = Map();
obj1Map.put("id",accID);
obj1Map.put("Add_Contacts",contList);
// info obj1Map;

objList = List();
objList.add(obj1Map);
multMap = Map();
multMap.put("data",objList);
info multMap;

resp = invokeurl
[
 	url :"https://www.zohoapis.in/crm/v8/Accounts/" + accID
 	type :PUT
 	parameters:multMap.toString()
 	connection:"getdata"
];

info resp;
```

Json Format -- Create Quotes and Show in Deals --


```json
{
  "data": [
    {
      "id": "1136779000000569096",
      "Add_Contacts": [
        {
          "Add_Contacts_1": {
            "id": "1136779000000569290"
          }
        },
        {
          "Add_Contacts_1": {
            "id": "1136779000000572469"
          }
        }
      ]
    }
  ]
}
```


---

# Find Duplicate in Leads and Contacts Module and Add in Multi-Pickup List ( when lead is created ) :


```javascript
leadId = "1147162000000641141";
getLeadData = zoho.crm.getRecordById("Leads",leadId);
leadMobile = getLeadData.getJson("Mobile");
leadPhone = getLeadData.getJson("Phone");
duplicateName = "";
// Check Lead Duplicate...
searchLead = zoho.crm.searchRecords("Leads", 
   "(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");	
	
leadSize = searchLead.size();

if(leadSize > 1){
	duplicateName ="Lead Duplicate";
	info duplicateName;
}else{
	duplicateName = "No Duplicate";
	info duplicateName;
}

// Check Contact Duplicate...

searchContact = zoho.crm.searchRecords("Contacts", 
   "(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");	

   if(searchContact.isEmpty()){
		info duplicateName;	
}else{

	if(duplicateName == "Lead Duplicate"){
		duplicateName = "Lead & Contact Duplicate";
		info duplicateName;
	}else{
		duplicateName = "Contact Duplicate";
		info duplicateName;
	}
	
}

if(duplicateName == "Lead Duplicate"){
	update = zoho.crm.updateRecord("Leads", leadId, {"Duplicate":duplicateName});
	info update;
}
else if(duplicateName == "Contact Duplicate"){
	update = zoho.crm.updateRecord("Leads", leadId, {"Duplicate":duplicateName});
	info update;
}
else if(duplicateName == "Lead & Contact Duplicate"){
	update = zoho.crm.updateRecord("Leads", leadId, {"Duplicate":duplicateName});
	info update;
}
```

Json Format -- for Adding in Multi-Pickup List --


```json
{"Duplicate":duplicateName}
```


---

# Add in Multi-Select Lookup in AccountsXContacts:

```javascript
 	accId = "1136779000000726001";
 	cont1 = "1136779000000569289";
 	cont2 = "1136779000000569290";
 
 	contList = List();
 	contList.add(cont1);
 	contList.add(cont2);
 
 	res=zoho.crm.getRecords("Accounts_X_Contacts");
 	info res;

 	for each cont in contList
 	{
 		mp=map();
 		mp.put("add_contacts",accId);
 		mp.put("Add_Contacts_1",cont);
 		create=zoho.crm.createRecord("Accounts_X_Contacts", mp);
 		info create;
 	}
```


Json Format -- for Updating in Multi - Lookpup in AccountsXContacts --


```json
{"add_contacts":"1136779000000726001", // accId
"Add_Contacts_1":"1136779000000569290"} // lookupId from getRecords linking module
```


---

# Auto Folder Creation In WorkDrive For Deals :

```javascript
dealId = "1234567890";
folderId = "jw0g37ef3b4f25b514cecabb2c5c16d785023";
header = Map();
header.put("Accept","application/vnd.api+json");
attMap = Map();
attMap.put("name",dealId);
attMap.put("parent_id",folderId);
// info attMap;

wrapMap = Map();
wrapMap.put("attributes",attMap);
wrapMap.put("type","files");
// info wrapMap;

dataMap = Map();
dataMap.put("data",wrapMap);
// info dataMap;

response = invokeurl
[
 	url :" https://www.zohoapis.in/workdrive/api/v1/files"
 	type :POST
 	parameters:dataMap.toString()
 	headers:header
 	connection:"workdrive"
];

info response;
newFolderId = response.get("data").get("attributes").get("permalink");
// info newFolderId;

urlMap = Map();
urlMap.put("URL",newFolderId);
// 	info urlMap;

updateDeal = zoho.crm.updateRecord("Deals",dealId,urlMap);
info updateDeal;
```

Json Format -- for autoFolderCreationInWorkDriveForDeals --


```json
{
  "data": {
    "attributes": {
      "name": 1234567890,
      "parent_id": "jw0g37ef3b4f25b514cecabb2c5c16d785023"
    },
    "type": "files"
  }
}
```


---

# Add in Multi-Select Lookup in AccountsXUsers:

```javascript
 userId = "1136779000000426001";

 // info zoho.crm.getRecords("Accounts_X_Users");

 mp=map();
 mp.put("userlookup221_3",accID);
 mp.put("User",userId);
 create=zoho.crm.createRecord("Accounts_X_Users", mp);
 info create;

```

Json Format -- for Updating in Multi - Lookpup in AccountsXContacts --


```json
{"userlookup221_3":"1136779000000569096", // accId
"User":"1136779000000426001"}
```


---

# Upload files from Attachments in Deal Record to WorkDrive :


```javascript
// 	dealId = "1136779000000758081";
workdriveFolderId = "jw0g37ef3b4f25b514cecabb2c5c16d785023";
fileName = "delugeRemainder.txt";
header = Map();
header.put("Accept","application/vnd.api+json");
getFileData = zoho.crm.getRelatedRecords("Attachments","Deals",dealId);
//     info res;
for each  data in getFileData
{
 	file = invokeurl
 	[
 		url :data.get("$link_url")
 		type :GET
 	];
 	// 	info file;
 	list_of_text = List();
 	list_of_text.add({"paramName":"filename","content":data.get("File_Name"),"stringPart":"true"});
 	list_of_text.add({"paramName":"parent_id","content":workdriveFolderId,"stringPart":"true"});
 	list_of_text.add({"paramName":"override-name-exist","content":"true","stringPart":"true"});
 	list_of_text.add({"paramName":"content","content":file,"stringPart":"false"});
 	// 	info list_of_text;
 	response = invokeurl
 	[
 		url :"https://www.zohoapis.in/workdrive/api/v1/upload"
 		type :POST
 		headers:header
 		files:list_of_text
 		connection:"workdrive"
 	];
 	info response.get("data").getJson("attributes").get("Permalink");
}

return "";
```

Json Format -- for Uplad File In WorkDrive From Attachments in Deals --


```json
[
  {
    "paramName": "filename",
    "content": "delugeRemainder.txt",
    "stringPart": "true"
  },
  {
    "paramName": "parent_id",
    "content": "jw0g37ef3b4f25b514cecabb2c5c16d785023",
    "stringPart": "true"
  },
  {
    "paramName": "override-name-exist",
    "content": "true",
    "stringPart": "true"
  },
  {
    "paramName": "content",
    "content": "<!DOCTYPE html></body></html>",
    "stringPart": "false"
  }
]
```


---

# Update Tag -- It add new tag & remove old tag : 


```javascript
dealId = "1136779000000758081";

// Tags that will replace all existing tags on the deal
replacementTags = List();

tag1 = Map();
tag1.put("name","POP");
replacementTags.add(tag1);

tag2 = Map();
tag2.put("name","POPPY");
replacementTags.add(tag2);

updateMap = Map();
updateMap.put("Tag", replacementTags);

info updateMap;

update = zoho.crm.updateRecord("Deals", dealId, updateMap);
info update;
```

Json Format -- Update Tag --


```json
{
  "Tag": [
    {
      "name": "POP"
    },
    {
      "name": "POPPY"
    }
  ]
}
```


---

# Add a new Tag and keep the existing one's :

```javascript
dealId = "1136779000000758081";
	getDealData = zoho.crm.getRecordById("Deals", dealId);
	dealTag = getDealData.get("Tag");
	if(dealTag == null)
	{
	  dealTag = List();
	}

	newTag = Map();
	newTag.put("name","POP");
	dealTag.add(newTag);
	
	newTag1 = Map();
	newTag1.put("name","POPPY");
	dealTag.add(newTag1);
	
	updateMap = Map();
	updateMap.put("Tag",dealTag);
// info updateMap;
	update = zoho.crm.updateRecord("Deals", dealId, updateMap);
	info update;
```

Json Format -- 


```json
{
  "Tag": [
    {
      "name": "TTUYTU",
      "id": "1136779000000802459"
    },
    {
      "name": "GDFGD",
      "id": "1136779000000802461"
    },
    {
      "name": "YUYJH",
      "id": "1136779000000802509"
    },
    {
      "name": "POP"
    },
    {
      "name": "POPPY"
    }
  ]
}
```

---

# Send Mail using Deluge :

```javascript
### Method 1 (Without Template) - 

sendmail
[
	from :zoho.loginuserid
	to :recipientEmail
	subject :"Welcome to Our Service"
	message :"Hello " + "This is from test mail" + ",<br><br>Thank you for reaching out!"
	content type :HTML
]

### Method 1 (With Template) -

recipientEmail = "akashxwork@gmail.com";
templateId = "1136779000000857005";

templatedata = invokeurl
[
	url :"https://www.zohoapis.in/crm/v8/settings/email_templates/" + templateId
	type :GET
	connection:"getdata"
];

// info templatedata;
templatecontent = templatedata.get("email_templates").get("0").get("content");
// info templatecontent;

LeadName = "Akhri Pasta";
Email = recipientEmail;
revisedcontent = templatecontent.replaceAll("\$\{!Leads.Full_Name\}",LeadName).replaceAll("\$\{!Leads.Email\}",Email);
// info revisedcontent;

sendmail
[
from: zoho.adminuserid
to: recipientEmail
subject: "Welcome Email Template"
message: revisedcontent
]

### Method 2 (With Template) -

id = "1234567890";
template_id = "920302000014877634";
FromEmail = "fromtest@mail.com";
toEmail = "totestmail.com";

url = "https://www.zohoapis.in/crm/v8/OA_Records/" + id + "/actions/send_mail";

data = {"data":{{"from":{"email":FromEmail},"to":{{"email":toEmail}},"template":{"id":template_id}"org_email":true,"related_entity_id":id,"module":"OA_Records"}}};

response = invokeurl
[
	url :url
	type :POST
	parameters:data.toString()
	connection:"sendemailtemplate"
];
	
info response;
```


---

# File Upload in Upload File ( Field ) :

```javascript
Note - Get file from attachments and add in file upload field in leads.

leadId = "1136779000000569198"; 

att = zoho.crm.getRelatedRecords("Attachments", "Leads", leadId);

file = invokeurl
[
	url: "https://www.zohoapis.in/crm/v2/Leads/" + leadId + "/Attachments/" + att.get(0).get("id")
	type: GET
	connection:"zohofile"
];
// info file;

file.setParamName("file");

resp = invokeurl 
[ 
	url: " https://www.zohoapis.in/crm/v2/files" 
	type: POST 
	files: file 
	connection: "zohofile" 
]; 
// info resp;

fileid = resp.get("data").get(0).get("details").get("id"); 
// info fileid;

fmp = Map(); 
fmp.put("file_id",fileid); 
fileList = List();
fileList.add(fmp); 
mp = Map(); 
mp.put("File_Upload",fileList); 
update = zoho.crm.updateRecord("Leads", leadId, mp);
info update;

```


---

# File Upload in Attachments :

```javascript
Note - Get file from upload file field and add in attachments in leads.

leadId = "1136779000000569198";
leadRecord = zoho.crm.getRecordById("Leads", leadId);
attId = leadRecord.get("File_Upload").getJSON("attachment_Id");
// info attId;

file = invokeurl
[
url :"https://www.zohoapis.in/crm/v2/Leads/" + leadId + "/Attachments/" + attId
type :GET
connection:"zohofile"
];

resp = zoho.crm.attachFile("Leads", leadId, file);
info resp;

```


---

# Create Quotes and Show in Deals :

```javascript
Note - Take from Quotes Items & add products in same Deal Records(Both are Related List from Deal Record).

	dealId = "1136779000000569369";
	
	getProductData = zoho.crm.getRelatedRecords("Quotes", "Deals", dealId);
// 	info getProductData;
	products = getProductData.getJson("Product_Details");
// 	info products;
	
	
	productList = List();
	
	for each product in products
    {
		productId = product.get("product").get("id");
		mp = Map();
		update = zoho.crm.updateRelatedRecord("Products", productId, "Deals", dealId,mp);
		info update; 
    }
```


---

# Auto Age Calculate :

```javascript
// extract DOB
dobDay = 10;
dobMonth = 1;
dobYear = 2000;
now = zoho.currenttime;
// info now;
// extract current time
currentYear = now.getYear();
currentMonth = now.getMonth();
currentDay = now.getDay();
currentHour = now.getHour();
currentMinute = now.getMinutes();
currentSecond = now.getSeconds();
// calculate time
year = currentYear - dobYear;
month = currentMonth - dobMonth;
day = currentDay - dobDay;
if(month < 0)
{
	year = year - 1;
	month = month + 12;
}
if(day < 0)
{
	month = month - 1;
	if(month < 0)
	{
		year = year - 1;
		month = month + 12;
	}

	prevMonth = currentMonth - 1;
	if(prevMonth == 0)
	{
		prevMonth = 12;
	}

	daysInMonth = {1:31,2:28,3:31,4:30,5:31,6:30,7:31,8:31,9:30,10:31,11:30,12:31};
	day = day + daysInMonth.get(prevMonth);
}

age = year + "y " + month + "m " + day + "d " + " | " + currentHour + ":" + currentMinute + ":" + currentSecond;
info age;
```


---

# Create Invoice and Show in Deals(Transaction) :

```javascript
Note - Take Product which is shares(Related List) from Deal Record & create invoices(Same from Related List from Products - Shares).

// info zoho.crm.getRelatedRecords("Invoices", "Deals", "7171648000000613057");
// info zoho.crm.getRecordById("Invoices","7171648000000613109");
	
dealId = "7171648000000601064";
getDealData = zoho.crm.getRecordById("Deals",dealId);
shareName = getDealData.get("Share_Name");
// info getDealData;

if( shareName == null){
shareId = getDealData.get("Share_Name").get("id");
shareQuantity = getDealData.get("Share_Quantity");
sellRate = getDealData.get("Sell_Rate");
contactId = getDealData.get("Contact_Name").get("id");

getContactData = zoho.crm.getRecordById("Contacts",contactId);

contactCountry = getContactData.get("Mailing_Country");
contactState = getContactData.get("Mailing_State");
contactCity = getContactData.get("Mailing_City");
contactStreet = getContactData.get("Mailing_Street");
// info contactStreet;

shareList = List();
shareMap = Map();
shareMap.put("product",{"id":shareId});
shareMap.put("quantity",shareQuantity);
shareMap.put("unit_price",sellRate);
shareList.add(shareMap);
info shareList;

mp = Map();
mp.put("Subject", "Sample");
mp.put("Deal_Name__s", dealId);
mp.put("Product_Details", shareList);
mp.put("Billing_Street",contactStreet);
mp.put("Billing_City",contactCity);
mp.put("Billing_State",contactState);
mp.put("Billing_India",contactCountry);
mp.put("Shipping_Street",contactStreet);
mp.put("Shipping_City",contactCity);
mp.put("Shipping_State",contactState);
mp.put("Shipping_India",contactCountry);
info mp;

res = zoho.crm.createRecord("Invoices", mp);
info res;
}
else{
	info "nahi hoga";
}
```

Json Format -- Create Invoice and Show in Deals (Transaction) --

```json
{
  "Subject": "Sample",
  "Deal_Name__s": "7171648000000601064",
  "Product_Details": [
    {
      "product": {
        "id": "7171648000000610040"
      },
      "quantity": 90,
      "unit_price": 9876
    }
  ],
  "Billing_Street": "228 Runamuck Pl #2808",
  "Billing_City": "Baltimore",
  "Billing_State": "MD",
  "Billing_India": "United States",
  "Shipping_Street": "228 Runamuck Pl #2808",
  "Shipping_City": "Baltimore",
  "Shipping_State": "MD",
  "Shipping_India": "United States"
}
```


---

# Create a Duplicate List in Related List using Related Function in Leads (part-1):

```javascript
leadId = "1158761000000572119";
getLeadData = zoho.crm.getRecordById("Leads",leadId);
leadMobile = getLeadData.getJson("Mobile");
leadPhone = getLeadData.getJson("Phone");

// Check Lead Duplicate...
searchLead = zoho.crm.searchRecords("Leads","(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");
info searchLead;

// Check Contact Duplicate...
searchContact = zoho.crm.searchRecords("Contacts","(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");
info searchContact;

if(searchLead.size() > 1 || searchContact.size() > 0)
{
	relatedListXML = "<record>";
	n = 0;
	for each  leadRecord in searchLead
	{
		if(leadRecord.get("id") != leadId)
		{
			leadName = leadRecord.get("Full_Name");
			leadPhone = leadRecord.get("Mobile");
			relatedListXML = relatedListXML + "<row no='" + n + "'><FL val='Lead Name'>" + leadName + "</FL><FL val='Phone'>" + leadPhone + "</FL></row>";
			n = n + 1;
		}
	}
	for each  contactRecord in searchContact
	{
		contactName = contactRecord.get("Full_Name");
		contactPhone = contactRecord.get("Mobile");
		relatedListXML = relatedListXML + "<row no='" + n + "'><FL val='Contact Name'>" + contactName + "</FL><FL val='Phone'>" + contactPhone + "</FL></row>";
		n = n + 1;
	}
	relatedListXML = relatedListXML + "</record>";
}

return relatedListXML;
```

XML format --

```javascript
<record>
	<row no='0'>
		<FL val='Lead Name'>Sharma</FL>
		<FL val='Phone'>9876543210</FL>
	</row>
	<row no='1'>
		<FL val='Contact Name'>Sharma Ji</FL>
		<FL val='Phone'>9876543210</FL>
	</row>
	<row no='2'>
		<FL val='Contact Name'>Verma Ji</FL>
		<FL val='Phone'>9876543210</FL>
	</row>
</record>
```


---

# Remove all alphabet & special character using regex :

```javascript
	number = "$%^#$336356sjkdf456";
	info number.replaceAll("[^0-9]", "");


```


---

# Create a Duplicate List in Related List using Related Function in Leads (part-2) :

```javascript
string related_list.create_list_in_leads(String leadId)
{
// leadId = "1158761000000572119";
getLeadData = zoho.crm.getRecordById("Leads",leadId);
// info getLeadData;
leadMobile = getLeadData.getJson("Mobile");
leadPhone = getLeadData.getJson("Phone");
// Check Lead Duplicate...
searchLead = zoho.crm.searchRecords("Leads","(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");
// info searchLead;
// Check Contact Duplicate...
searchContact = zoho.crm.searchRecords("Contacts","(Phone:equals:" + leadPhone + ") or (Mobile:equals:" + leadPhone + ") or (Phone:equals:" + leadMobile + ") or (Mobile:equals:" + leadMobile + ")");
info searchContact;
if(searchLead.size() > 1 || searchContact.size() > 0)
{
	relatedListXML = "<record>";
	// XML for Lead Category
	n = 0;
	for each  leadRecord in searchLead
	{
		if(leadRecord.get("id") != leadId)
		{
			lead_name = leadRecord.get("Full_Name");
			lead_phone = ifnull(leadRecord.get("Mobile"),leadRecord.get("Phone"));
			lead_id = leadRecord.get("id");
			lead_owner = leadRecord.get("Owner").get("name");
			row = "<row no='" + n + "'>";
			flName = "<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/" + lead_id + "'>" + lead_name + "</FL>";
			flRecord = "<FL val='Record'>Lead</FL>";
			flPhone = "<FL val='Phone'>" + lead_phone + "</FL>";
			flOwner = "<FL val='Owner'>" + lead_owner + "</FL>";
			relatedListXML = relatedListXML + row + flName + flPhone + flRecord + flOwner + "</row>";
			n = n + 1;
		}
	}
	// XML for Contact Category
	for each  contactRecord in searchContact
	{
		contact_name = contactRecord.get("Full_Name");
		contact_phone = ifnull(contactRecord.get("Mobile"),contactRecord.get("Phone"));
		contact_id = contactRecord.get("id");
		contact_owner = leadRecord.get("Owner").get("name");
		row = "<row no='" + n + "'>";
		flName = "<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/" + contact_id + "'>" + contact_name + "</FL>";
		flRecord = "<FL val='Record'>Contact</FL>";
		flPhone = "<FL val='Phone'>" + contact_phone + "</FL>";
		flOwner = "<FL val='Owner'>" + contact_owner + "</FL>";
		relatedListXML = relatedListXML + row + flName + flPhone + flRecord + flOwner + "</row>";
		n = n + 1;
	}
	relatedListXML = relatedListXML + "<row no='" + n + "'><FL val='Name'> Total - " + n + "</FL></row></record>";
	// info relatedListXML;
}
else
{
	relatedListXML = "<error><message>No Duplicate Contact</message></error>";
}
return relatedListXML;
}
```


XML format for Duplicate List in Related List --


```javascript
<record>
	<row no='0'>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/1158761000000593555'>Sharma</FL>
		<FL val='Record'>Lead</FL>
		<FL val='Phone'>9876543210</FL>
		<FL val='Owner'>Akash</FL>
	</row>
	<row no='1'>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/1158761000000617742'>Sharma Ji</FL>
		<FL val='Record'>Contact</FL>
		<FL val='Phone'>9876543210</FL>
		<FL val='Owner'>Akash</FL>
	</row>
	<row no='2'>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/1158761000000593575'>Verma Ji</FL>
		<FL val='Record'>Contact</FL>
		<FL val='Phone'>9876543210</FL>
		<FL val='Owner'>Akash</FL>
	</row>
	<row no='3'>
		<FL val='Name'> Total - 3</FL>
	</row>
</record>


```


---

# Create a Duplicate List in Related List using Related Function in Leads (part-3):

```javascript
leadData = zoho.crm.getRecordById("Leads",leadId);
if(leadData.containsKey("Mobile") && leadData.get("Mobile") != null)
{
	mobile = leadData.get("Mobile");
	search_lead = zoho.crm.searchRecords("Leads","(Mobile:equals:" + mobile + ")");
	// 	info search_lead;
	search_contact = zoho.crm.searchRecords("Contacts","(Mobile:equals:" + mobile + ")");
	// 	info contacts_data;
	if(search_lead.size() > 1 || search_contact.size() > 0)
	{
		relatedListXML = "<record>";
		count = 0;
		lead_count = 0;
		contact_count = 0;
		for each  leadRecord in search_lead
		{
			if(leadRecord.get("id") != leadId)
			{
				
				lead_name = leadRecord.get("Full_Name");
				lead_phone = ifnull(leadRecord.get("Mobile"),leadRecord.get("Phone"));
				lead_id = leadRecord.get("id");
				lead_owner = leadRecord.get("Owner").get("name");
				lead_status = leadRecord.get("Lead_Status");
				row = "<row no='" + count + "'>";
				flName = "<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/" + lead_id + "'>" + lead_name + "</FL>";
				flRecord = "<FL val='Record'>Lead</FL>";
				flPhone = "<FL val='Phone'>" + lead_phone + "</FL>";
				flOwner = "<FL val='Owner'>" + lead_owner + "</FL>";
				flStatus = "<FL val='Status'>" + lead_status + "</FL>";
				relatedListXML = relatedListXML + row + flRecord + flName + flPhone + flOwner + flStatus + "</row>";
				lead_count = lead_count + 1;
				count = count + 1;
			}
		}
		for each  contactRecord in search_contact
		{
			
			contact_name = contactRecord.get("Full_Name");
			contact_phone = ifnull(contactRecord.get("Mobile"),contactRecord.get("Phone"));
			contact_id = contactRecord.get("id");
			contact_owner = contactRecord.get("Owner").get("name");
			row = "<row no='" + count + "'>";
			flName = "<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/" + contact_id + "'>" + contact_name + "</FL>";
			flRecord = "<FL val='Record'>Contact</FL>";
			flPhone = "<FL val='Phone'>" + contact_phone + "</FL>";
			flOwner = "<FL val='Owner'>" + contact_owner + "</FL>";
			flStatus = "<FL val='Status'>—————</FL>";
			relatedListXML = relatedListXML + row + flRecord + flName + flPhone + flOwner + flStatus + "</row>";
			contact_count = contact_count + 1;
			count = count + 1;
		}
// 		relatedListXML = relatedListXML + "<row no='" + count + "'><FL val='Name'> Total - " + count + "</FL></row></record>";
		relatedListXML = relatedListXML + "<row no='" + count + "'><FL val='Name'> Lead - " + lead_count + " | " + "Contact - " + contact_count + "</FL></row></record>";
	}
	else
	{
		relatedListXML = "<error><message>No matching records found.</message></error>";
	}
	return relatedListXML;
}
else
{
	return "<error><message>No matching records found.</message></error>";
}
```


XML format --

```javascript
<record>
	<row no='0'>
		<FL val='Record'>Lead</FL>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/991021000006306009'>test 1 L</FL>
		<FL val='Phone'>919876543210</FL>
		<FL val='Owner'>Sales_IN</FL>
		<FL val='Status'>Fresh Lead</FL>
	</row>
	<row no='1'>
		<FL val='Record'>Lead</FL>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Leads/991021000006306051'>test 2 L</FL>
		<FL val='Phone'>919876543210</FL>
		<FL val='Owner'>Sales_IN</FL>
		<FL val='Status'>Fresh Lead</FL>
	</row>
	<row no='2'>
		<FL val='Record'>Contact</FL>
		<FL val='Name' link='true' url='https://crm.zoho.in/crm/org60062439154/tab/Contacts/991021000006306093'>test 1 C</FL>
		<FL val='Phone'>919876543210</FL>
		<FL val='Owner'>Sales_IN</FL>
		<FL val='Status'>—————</FL>
	</row>
	<row no='3'>
		<FL val='Name'> Lead - 2 | Contact - 1</FL>
	</row>
</record>
```

---

# Some Helpers for using Related Function :

```javascript
// if there is "&" in string - it helps from breaking XML
acc_sub_segment = ifnull(accData.get("Sub_Segment"),"---").toString().replaceAll("&","&amp;");

// for better visibility 
url = "'https://crm.zoho.in/crm/org60040721686/tab/Accounts/" + account_id + "'";
flName = "<FL val='Account Name' link='true' url=" + url + ">" + account_name + "</FL>";
```

---

# Get Files from Workdrive and Show in Related List using XML  (part-1) :


```javascript
string related_list.preview_workdrive_attachments(String leadId)
{
workdriveFolderId = "lpjr97c99675b349f4fa9bca794815e2defae";
header = Map();
header.put("Accept","application/vnd.api+json");
response = invokeurl
[
url :"https://www.zohoapis.in/workdrive/api/v1/files/" + workdriveFolderId + "/files"
type :GET
headers:header
connection:"workdrive"
];
files = response.get("data");
// info files.get("0").get("attributes") ;
if(files.size() > 0)
{
relatedListXML = "<record>";
count = 0;
for each file in files
{
name = file.get("attributes").get("name");
fileName = name.substring(0, name.lastIndexOf("."));
// info fileName;
fileId = file.get("id");
filePermalink = file.get("attributes").get("permalink");
fileDownloadURL = file.get("attributes").get("download_url");
filePreview = "https://workdrive.zoho.in/preview/" + fileId;
fileType = file.get("attributes").get("type");
fileSize = file.get("attributes").get("storage_info").get("size");

row = "<row no='" + count + "'>";
flName = "<FL val='File Name' link='true' url='" + filePermalink + "'>" + fileName + "</FL>";
flPreview = "<FL val='' link='true' url='" + filePreview + "'>Preview</FL>";
flDownload = "<FL val='' link='true' url='" + fileDownloadURL + "'>Download</FL>";
flType = "<FL val='Type'>" + fileType + "</FL>";
flSize = "<FL val='Size'>" + fileSize + "</FL>";

relatedListXML = relatedListXML + row + flName + flPreview + flType + flSize + "</row>";
count = count + 1;
}
relatedListXML = relatedListXML + "</record>";
// info relatedListXML;
}
else
{
relatedListXML = "<error><message>No WorkDrive Files/Folders</message></error>";
}
return relatedListXML;
}
```


XML format for Duplicate List in Related List --

```javascript
<record>
	<row no='0'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/lpjr9383ca3ec09f641d9912ea2bae983caf3'>Man Search for Meaning</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/lpjr9383ca3ec09f641d9912ea2bae983caf3'>Preview</FL>
		<FL val='Type'>pdf</FL>
		<FL val='Size'>316.21 KB</FL>
	</row>
	<row no='1'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/lpjr91e0ed4f7399340fc9acd949dab22e60d'>delugeRemainder</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/lpjr91e0ed4f7399340fc9acd949dab22e60d'>Preview</FL>
		<FL val='Type'>docs</FL>
		<FL val='Size'>25.20 KB</FL>
	</row>
	<row no='2'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/a1kl8dea62e91ac7349e8ac819c4cf4072c37'>AaeraCampaignleads</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/a1kl8dea62e91ac7349e8ac819c4cf4072c37'>Preview</FL>
		<FL val='Type'>spreadsheet</FL>
		<FL val='Size'>298.64 KB</FL>
	</row>
	<row no='3'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/a1kl897e4e1b5c5094368a121c1999a07d154'>Aaeraleads2025</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/a1kl897e4e1b5c5094368a121c1999a07d154'>Preview</FL>
		<FL val='Type'>spreadsheet</FL>
		<FL val='Size'>3.08 MB</FL>
	</row>
	<row no='4'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/a1kl857fc2863c2b04155b806eca6a76d44d1'>Vitamin_Cheat_Sheet</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/a1kl857fc2863c2b04155b806eca6a76d44d1'>Preview</FL>
		<FL val='Type'>pdf</FL>
		<FL val='Size'>53.88 MB</FL>
	</row>
	<row no='5'>
		<FL val='File Name' link='true' url='https://workdrive.zoho.in/file/lpjr954f4b39f4076414aa87b1afcec819d4e'>The Monk Who Sold His Ferrari</FL>
		<FL val='' link='true' url='https://workdrive.zoho.in/preview/lpjr954f4b39f4076414aa87b1afcec819d4e'>Preview</FL>
		<FL val='Type'>pdf</FL>
		<FL val='Size'>1.25 MB</FL>
	</row>
</record>
```


---

# Get Files from Workdrive and Show in Related List using XML  (part-2) :

```javascript
// caseId = "1043751000008644045";
caseData = zoho.crm.getRecordById("Cases",caseId);
// 	info caseData.get("Contact_Data_Link");
url = caseData.get("Contact_Data_Link");
if(url == null || url.isEmpty()){
return "<error><message>No WorkDrive Files/Folders</message></error>";
}
workdriveFolderId = url.subString(url.lastIndexOf("/") + 1);
// 	info workdriveFolderId;
header = Map();
header.put("Accept","application/vnd.api+json");
response = invokeurl
[
	url :"https://www.zohoapis.in/workdrive/api/v1/files/" + workdriveFolderId + "/files"
	type :GET
	headers:header
	connection:"contactworkdrive"
];

files = response.get("data");
// info files;
if(files.size() > 0)
{
	relatedListXML = "<record>";
	count = 0;
	for each  file in files
	{
		name = file.get("attributes").get("name");
		fileName = name.substring(0,name.lastIndexOf("."));
		// info fileName;
		fileId = file.get("id");
		filePermalink = file.get("attributes").get("permalink");
		fileDownloadURL = file.get("attributes").get("download_url");
		filePreview = "https://workdrive.zoho.in/preview/" + fileId;
		fileSize = file.get("attributes").get("storage_info").get("size");
		fileExtn = file.get("attributes").get("extn");
		fileType = "";
		if(fileExtn == "zip")
		{
			fileType = fileExtn;
		}
		else
		{
			fileType = file.get("attributes").get("type");
		}
		row = "<row no='" + count + "'>";
		flName = "<FL val='File Name' link='true' url='" + filePermalink + "'>" + fileName + "</FL>";
		flPreview = "<FL val='' link='true' url='" + filePreview + "'>Preview</FL>";
		flDownload = "<FL val='' link='true' url='" + fileDownloadURL + "'>Download</FL>";
		flType = "<FL val='Type'>" + fileType + "</FL>";
		flSize = "<FL val='Size'>" + fileSize + "</FL>";
// 		flUpload = "<FL val='Action' link='true' url='" + url + "'>► UPLOAD FILE</FL>";
		relatedListXML = relatedListXML + row + flName + flPreview + flType + flSize + "</row>";
		count = count + 1;
	}
		relatedListXML = relatedListXML + "<row no='" + count + "'><FL val='Type'>Action</FL><FL val='Size' link='true' url='" + url + "'>⬆️ Click to Upload</FL></row></record>";
	// info relatedListXML;
}
else
{
	relatedListXML = "<error><message>No WorkDrive Files/Folders</message></error>";
}

return relatedListXML;
```


---

# Get Files from Workdrive and Show in Related List using XML  (part-3) :
```javascript
// dealId = "1043751000009630359";
dealData = zoho.crm.getRecordById("Deals",dealId);
// 	info dealData.get("Contact_Data_Link");
url = dealData.get("Contact_Data_Link");
// info url;
if(url == null || url.isEmpty())
{
	return "<error><message>Please attach a WorkDrive link before proceeding.</message></error>";
}
workdriveFolderId = url.subString(url.lastIndexOf("/") + 1);
// info workdriveFolderId;

header = Map();
header.put("Accept","application/vnd.api+json");
response = invokeurl
[
	url :"https://www.zohoapis.in/workdrive/api/v1/files/" + workdriveFolderId + "/files"
	type :GET
	headers:header
	connection:"contactworkdrive"
];

files = response.get("data");
// info files;
if(files.size() > 0)
{
	relatedListXML = "<record>";
	count = 0;
	for each  file in files
	{
		name = file.get("attributes").get("name");
		fileName = name.substring(0,name.lastIndexOf("."));
		// info fileName;
		fileId = file.get("id");
		filePermalink = file.get("attributes").get("permalink");
		fileDownloadURL = file.get("attributes").get("download_url");
		filePreview = "https://workdrive.zoho.in/preview/" + fileId;
		fileSize = file.get("attributes").get("storage_info").get("size");
		fileExtn = file.get("attributes").get("extn");
		fileType = "";
		if(fileExtn == "zip")
		{
			fileType = fileExtn;
		}
		else
		{
			fileType = file.get("attributes").get("type");
		}
		row = "<row no='" + count + "'>";
		flName = "<FL val='File Name' link='true' url='" + filePermalink + "'>" + fileName + "</FL>";
		flPreview = "<FL val='' link='true' url='" + filePreview + "'>Preview</FL>";
		flDownload = "<FL val='' link='true' url='" + fileDownloadURL + "'>Download</FL>";
		flType = "<FL val='Type'>" + fileType + "</FL>";
		flSize = "<FL val='Size'>" + fileSize + "</FL>";
		// flUpload = "<FL val='File Name' link='true' url='" + url + "'>Upload</FL>";
		relatedListXML = relatedListXML + row + flName + flPreview + flType + flSize + "</row>";
		count = count + 1;
	}
	relatedListXML = relatedListXML + "<row no='" + count + "'><FL val='Type'>Action</FL><FL val='Size' link='true' url='" + url + "'>⬆️ Click to Upload</FL></row></record>";
	// info relatedListXML;
}
else
{
		relatedListXML = "<record><row no='0'><FL val=''>No file uploaded yet</FL><FL val='' link='true' url='" + url + "'>⬆️ Click here to upload a file</FL></row></record>";
}

return relatedListXML;
```


---

# Mobile Validator in Lead using Layout Rule :

```javascript
entityMap = crmAPIRequest.toMap().get("record");
mobile = entityMap.get("Mobile");
response = Map();
// if(ifNull(mobile, "").matches("^[0-9]{10}$") == false) 
if(mobile != "" && mobile.matches("^[0-9]{10}$") == false)
{
	response.put('status','error');
	response.put('message','Please enter a valid 10-digit mobile number.');
	return response;
}

return "";
```


---

# If Closed Won Stage is change to another Stage, it will gives error using Layout Rule :

```javascript
entityMap = crmAPIRequest.toMap().get("record");
stage = ifNull(entityMap.get("Stage"),"");
recordID = entityMap.get("id");
response = Map();
if(recordID == null)
{
	response.put('status','success');
}
else if(stage != "Closed Won")
{
	response.put('status','error');
	response.put('message','Not allowed to change stage');
}
else
{
	response.put('status','success');
}

return response;
```


---

# Bulk Auto Update "Product Code" Field in Product Module :

```javascript
product_data = zoho.crm.getRecords("Products",0,200,{"cvid":785423000001339227});
for each  rec in product_data
{
	id = rec.get("id");
	// 	info id;
	Industry1 = rec.get("Industry1");
	Product_Category = rec.get("Product_Category");
	Product_Name = rec.get("Product_Name");
	Practice_Area = rec.get("Practice_Area");
	Sub_Practice_Area = rec.get("Sub_Practice_Area");
	Product_Code = Industry1 + "/" + Product_Category + "/" + Product_Name + "/" + Practice_Area + "/" + Sub_Practice_Area;
	Product_Code = Product_Code.replaceAll("/null","");
	Product_Code = Product_Code.replaceAll("null/","");
	res = zoho.crm.updateRecord("Products",id,{"Product_Code":Product_Code,"up":true});
}

```
---

# Auto Bulk Update ( 1000 record ) :

```javascript

cvid = "1234567890";
module = "Deals";
field_api = "Opportunity_Type";
checkbox_api = "Tests";

pages = {1, 2, 3, 4, 5}; 

for each pg in pages
{
    lead_data = zoho.crm.getRecords(module, pg, 200, {"cvid": cvid});
    
    if(lead_data.size() > 0)
    {
        for each rec in lead_data
        {
            rec_id = rec.get("id");
            res = zoho.crm.updateRecord(module, rec_id, {field_api : null, checkbox_api : true});
			info res;
        }
    }
    else {
        break;
    }
    info "Page No. : " + pg;
}
info "done";

```

---

# Update Status which have Blueprint transition using Deluge (Part - 1):

```javascript
caseId = "1043751000008644045";
	
response = invokeurl
[
	url: "https://www.zohoapis.in/crm/v8/Cases/" + caseId + "/actions/blueprint"
	type: GET
	connection:"zoho_crm"
];

// info response.get("blueprint").get("transitions");

dataMap = Map();
dataMap.put("Notes", "Application Submitted");

blueprint1 = Map();
blueprint1.put("transition_id", "1043751000008644280"); // Application Submitted

decisionDate = zoho.currentdate.toString("yyyy-MM-dd");
dataMap.put("Decision_Due_by", decisionDate);

blueprint1.put("data", dataMap);
blueprintList = List();
blueprintList.add(blueprint1);
param = Map();
param.put("blueprint", blueprintList);
info param;
update = invokeurl
[
	url: "https://www.zohoapis.in/crm/v8/Cases/" + caseId + "/actions/blueprint"
	type: PUT
	parameters: param.toString()
	connection:"zoho_crm"
];

info update;

```

Json Format --

```json
{
  "blueprint": [
    {
      "transition_id": "1043751000008644280",
      "data": {
        "Notes": "Application Submitted",
        "Decision_Due_by": "2026-01-13"
      }
    }
  ]
}
```


---

# Update Status to "Hold" which have Blueprint transition apart from "Visa Approved" or "Visa Rejected" using Deluge (Part - 2) :

```javascript
dealId = "1043751000007135808";
// 	caseId = "1043751000009263336";

// 	response = invokeurl
// [
// 	url: "https://www.zohoapis.in/crm/v8/Cases/" + caseId + "/actions/blueprint"
// 	type: GET
// 	connection:"zoho_crm"
// ];
// info response;
	
get_related_data = zoho.crm.getRelatedRecords("Cases", "Deals", dealId);

if(get_related_data.size() > 0)
{
	for each record in get_related_data
	{
		caseId = record.get("id");
		currentStatus = record.get("Status");

		if(currentStatus != "Visa Approved" && currentStatus != "Visa Rejected" && currentStatus != "Hold")
		{
			dataMap = Map();
			// dataMap.put("Notes", "Hold");
			blueprint1 = Map();
			blueprint1.put("transition_id", "1043751000009328223"); // Hold
			decisionDate = zoho.currentdate.toString("yyyy-MM-dd");
			dataMap.put("Decision_Due_by", decisionDate);
			blueprint1.put("data", dataMap);
			blueprintList = List();
			blueprintList.add(blueprint1);
			param = Map();
			param.put("blueprint", blueprintList);
			info param;
				res = invokeurl
			[
				url: "https://www.zohoapis.in/crm/v8/Cases/" + caseId + "/actions/blueprint"
				type: PUT
				parameters: param.toString()
				connection:"zoho_crm"
			];
			info res;
		}
	}
}
```


---

# Some Helpful Function :

```javascript
str = "one/two/three/four/four";
arr = str.toList("/");
arr.remove(arr.size() - 1);
arr = toString(arr);
arr = arr.replaceAll(",","/");
info arr;

number = "$%^#$33635328973326sjkdf456UJT";
info number.replaceAll("[^0-9]","");

num = "987654432101234";
if(num.length() > 10)
{
	num = num.subString(0,10);
}

info num;
```


---

# How to get Rollup Summary Field Value using Deluge :

```javascript
deal_id = "1158761000000660337";

response = invokeurl
[
url: "https://www.zohoapis.in/crm/v8/Deals/" + deal_id
type: GET
connection: "zohocrm"
];

info response.get("data").getJson("Rollup_Summary");

```


---

# Close all task in Case(application) inside an Deals Module :

```javascript
deal_id = "1043751000007110324";
deal_related = zoho.crm.getRelatedRecords("Cases","Deals",deal_id);
for each  case in deal_related
{
	status = case.getJSon("Status");
	app_related = zoho.crm.getRelatedRecords("Tasks","Cases",case.getJson("id"));
	if(status == "Hold" && app_related.size() > 0)
	{
		for each  rec in app_related
		{
			if(rec.get("Status") != "Completed")
			{
				update = zoho.crm.updateRecord("Tasks",rec.getJson("id"),{"Status":"Completed"});
				// info "task : " + update;
			}
		}
	}
}
```

---

# Create Multiple Deal with certain condition (Multiple Countries) :

```javascript
// Fetch deal record
dealRecord = zoho.crm.getRecordById("Deals", id);
countriesOfInterest = dealRecord.get("Country_of_Interest");
isFirstCountry = true;

// Check if multiple countries exist
if(countriesOfInterest.size() > 1)
{
    // Process each country
    for each country in countriesOfInterest
    {
        // Generate deal name components
        intakeDate = dealRecord.get("Possible_Intake");
        formattedMonthYear = intakeDate.toDate().toString("MMM yyyy");
        originalDealName = dealRecord.get("Deal_Name");
        updatedDealName = originalDealName + "/" + country + "/" + formattedMonthYear;
        
        // Create single-item country list
        singleCountryList = List();
        singleCountryList.add(country);
        
        if(isFirstCountry)
        {
            // Update existing deal record
            updateData = Map();
            updateData.put("Country_of_Interest", singleCountryList);
            updateData.put("Deal_Name", updatedDealName);
            updateResponse = zoho.crm.updateRecord("Deals", id, updateData);
            
            isFirstCountry = false;
        }
        else
        {
            // Clone existing deal record
            clonedDealData = dealRecord.toMap();
            
            // Remove system fields that shouldn't be copied
            clonedDealData.remove("id");
            clonedDealData.remove("Owner");
            clonedDealData.remove("Created_Time");
            clonedDealData.remove("Modified_Time");
            clonedDealData.remove("Created_By");
            clonedDealData.remove("Modified_By");
            clonedDealData.remove("Tag");
            
            // Create new deal record
            createResponse = zoho.crm.createRecord("Deals", clonedDealData);
            
            // Update the newly created deal
            if(createResponse.get("id") != null)
            {
                newDealId = createResponse.get("id");
                
                updateData = Map();
                updateData.put("Country_of_Interest", singleCountryList);
                updateData.put("Deal_Name", updatedDealName);
                updateResponse = zoho.crm.updateRecord("Deals", newDealId, updateData);
            }
        }
    }
}
else
{
    // Handle single country scenario
    currentDealName = dealRecord.get("Deal_Name");
    country = dealRecord.get("Country_of_Interest");
    intakeDate = dealRecord.get("Possible_Intake");
    formattedMonthYear = intakeDate.toDate().toString("MMM yyyy");
    updatedDealName = currentDealName + "/" + country + "/" + formattedMonthYear;
    
    // Update deal name only
    updateData = Map();
    updateData.put("Deal_Name", updatedDealName);
    updateResponse = zoho.crm.updateRecord("Deals", id, updateData);
}
```


---

# Update URL from Contact to Deal to Application(Case) Module if there is not link in contact then it will create :

```javascript
parentFolderId = "mp021cbba2181fc654b57b77c7a24cd2e87dd";
// dealId = "1043751000007135808";
caseData = zoho.crm.getRelatedRecords("Cases","Deals",dealId);
dealData = zoho.crm.getRecordById("Deals",dealId);
dealURL = dealData.get("Contact_Data_Link");

isUpdated = false;

if(dealURL != null && !dealURL.isEmpty())
{
	if(caseData != null && caseData.size() > 0)
	{
		// isUpdated = false;

		for each caseRec in caseData
		{
			caseId = caseRec.get("id");
			caseLink = caseRec.get("Contact_Data_Link");

			if(caseLink == null || caseLink.isEmpty())
			{
				updateMap = Map();
				updateMap.put("Contact_Data_Link",dealURL);
				zoho.crm.updateRecord("Cases",caseId,updateMap);
				isUpdated = true;
			}
		}

		if(isUpdated)
		{
			return "Url attached to Application Record (Related List)";
		}
		else
		{
			return "Application already has Contact Data Link";
		}
	}
	else
	{
		return "No Application found for this Deal";
	}
}	

contactNameFromDeal = dealData.get("Contact_Name");
if(contactNameFromDeal == null)
{
	return "No Contact linked to Deal";
}

contactId = contactNameFromDeal.get("id");
contactData = zoho.crm.getRecordById("Contacts",contactId);
contactName = contactData.get("Full_Name");
contactURL = contactData.get("Contact_Data_Link1");

if(contactURL != null && !contactURL.isEmpty())
{
// update deal "Contact_Data_Link"....
	updateDealMap = Map();
	updateDealMap.put("Contact_Data_Link",contactURL);
	zoho.crm.updateRecord("Deals",dealId,updateDealMap);
// update contact "Contact_Data_Link1"....
	if(caseData != null && caseData.size() > 0)
	{
		for each caseRec in caseData
		{
			caseId = caseRec.get("id");
			caseLink = caseRec.get("Contact_Data_Link");

			if(caseLink == null || caseLink.isEmpty())
			{
				updateMap = Map();
				updateMap.put("Contact_Data_Link",contactURL);
				zoho.crm.updateRecord("Cases",caseId,updateMap);
			}
		}
		return "Contact module link synced to Deal & Application :- " + contactURL;
	}
	else
	{
		return "Contact module link synced to Deal :- " + contactURL;
	}
}
else
{
	// create folder in workdrive...
	folderName = contactName + "_" + contactId;
	createFolderResp = zoho.workdrive.createFolder(folderName,parentFolderId,"contactworkdrive");
	if(!createFolderResp.containKey("data"))
	{
		return "Something went wrong";
	}
	folderPermalink = createFolderResp.get("data").get("attributes").get("permalink");
	// 	info folderPermalink;
	// update contact "Contact_Data_Link1"....
	updateContactMap = Map();
	updateContactMap.put("Contact_Data_Link1",folderPermalink);
	zoho.crm.updateRecord("Contacts",contactId,updateContactMap);
	// update deal "Contact_Data_Link....
	updateDealMap = Map();
	updateDealMap.put("Contact_Data_Link",folderPermalink);
	zoho.crm.updateRecord("Deals",dealId,updateDealMap);
	// update application "Contact_Data_Link....
	if(caseData != null && caseData.size() > 0)
	{
		for each  caseRec in caseData
		{
			caseId = caseRec.get("id");
			caseLink = caseRec.get("Contact_Data_Link");
			if(caseLink == null || caseLink.isEmpty())
			{
				updateMap = Map();
				updateMap.put("Contact_Data_Link",folderPermalink);
				zoho.crm.updateRecord("Cases",caseId,updateMap);
			}
		}
	}
	return "Url attached to Contact, Deal & Application";
}
```


---

# Create a new call record using deluge (Part - 1):

```javascript
number = "+917987322610";
dateString = "13/01/2026";
timeString = "17:16:50";
dateValue = dateString.toDate("dd/MM/yyyy");
dateTimeString = dateString + " " + timeString;
dateTimeValue = dateTimeString.toTime("dd/MM/yyyy HH:mm:ss");
isoDateTime = dateTimeValue.toString("yyyy-MM-dd'T'HH:mm:ssXXX");
mp = Map();
mp.put("Subject","Missed call from " + number);
mp.put("Call_Type","Missed");
// mp.put("Call_Duration","0:31");
mp.put("$se_module","Leads");
mp.put("Call_Status","Missed");
mp.put("Dialled_Number",number);
mp.put("Call_Start_Time",isoDateTime);
// mp.put("","");
create = zoho.crm.createRecord("Calls", mp);
info create;

```


---

# Create a new call record using deluge (Part - 2):

```javascript
mobile = "+911234567890";
isoDateTime = now.toString("yyyy-MM-dd'T'HH:mm:ssXXX");

dataMap = Map();
dataMap.put("Subject", "Missed call from " + mobile);
dataMap.put("Call_Type", "Missed");
dataMap.put("Call_Status", "Missed");
dataMap.put("Dialled_Number", mobile);
dataMap.put("Call_Start_Time", isoDateTime);
dataMap.put("To_Number__s", mobile);

dataList = List();
dataList.add(dataMap);

payload = Map();
payload.put("data", dataList);

response = invokeurl
[
    url :"https://www.zohoapis.in/crm/v8/Calls"
    type : POST
    parameters : payload.toString()
	connection : "zohocrm"
];

info response;

```

---

# Auto Lead Conversion in Contacts, Acocunts and Deals Module :

```javascript
// leadId = "6103868000006230215";
lead = zoho.crm.getRecordById("Leads",leadId);
if(lead.get("Lead_Status") == "Qualified")
{
	// For Contact
	contactMap = Map();
	contactMap.put("First_Name",lead.get("First_Name"));
	contactMap.put("Last_Name",lead.get("Last_Name"));
	contactMap.put("Email",lead.get("Email"));
	contactMap.put("Mobile",lead.get("Mobile"));
	contactMap.put("Mailing_Street",lead.get("Street"));
	contactMap.put("Mailing_City",lead.get("City"));
	contactMap.put("Mailing_State",lead.get("State"));
	contactMap.put("Mailing_Zip",lead.get("Zip_Code"));
	contactMap.put("Mailing_Country",lead.get("Country"));

	// 	For Account
	accountMap = Map();
	accountMap.put("Account_Name",ifnull(lead.get("Company"),lead.get("Full_Name")));
	accountMap.put("Email",lead.get("Email"));
	accountMap.put("Mobile",lead.get("Mobile"));
	accountMap.put("Billing_Street",lead.get("Street"));
	accountMap.put("Billing_City",lead.get("City"));
	accountMap.put("Billing_State",lead.get("State"));
	accountMap.put("Billing_Zip",lead.get("Zip_Code"));
	accountMap.put("Billing_Country",lead.get("Country"));

	// For Deal
	dealMap = Map();
	dealMap.put("Deal_Name",lead.get("Last_Name"));
	dealMap.put("Stage","Qualification");

	// Conversion
	convertMap = Map();
	convertMap.put("Accounts",accountMap);
	convertMap.put("Contacts",contactMap);
	convertMap.put("Deals",dealMap);
	
	convert = zoho.crm.convertLead(leadId,convertMap);
	info convert;
}

```

---

# Search and Associate - Missed Call Associator
Note: If there is not mobile number in Lead record then just create it and associate with Call Record, otherwise just associate (Call Record) with existing lead record.

```javascript

// id = "991021000006782255";
call_data = invokeurl
[
	url :"https://www.zohoapis.in/crm/v8/Calls/" + id
	type :GET
	connection:"zoho_crm"
];

call_type = call_data.get("data").getJson("Call_Type");
related_to = call_data.get("data").getJson("What_Id");
call_number = call_data.get("data").getJson("To_Number__s");
// call_owner = call_data.get("data").getJson("Owner").get("id");
info call_data;

if(call_number != null && call_number != "")
{
	if(call_number.startsWith("91"))
	{
		// do nothing
	}
	else if(call_number.startsWith("0"))
	{
		call_number = "91" + call_number.substring(1);
	}
	else
	{
		call_number = "91" + call_number;
	}
}

if(call_type == "Missed" && related_to == null)
{
	criteria = "(Mobile:equals:" + call_number + ")";
	search_lead = zoho.crm.searchRecords("Leads",criteria);
	// info search_lead.size();
	if(search_lead.size() > 0)
	{
		lead_id = null;
		for each  lead_rec in search_lead
		{
			lead_status = lead_rec.get("Lead_Status");
			if(lead_status != "Junk Lead")
			{
				lead_id = lead_rec.get("id");
				owner_id = lead_rec.get("Owner").get("id");
				break;
			}
		}
		if(lead_id == null)
		{
			lead_id = search_lead.get(0).get("id");
			owner_id = search_lead.get(0).get("Owner").get("id");
		}

		update_related_to = zoho.crm.updateRecord("Calls",id,{"$se_module":"Leads","What_Id":lead_id});
		info "if";

		// Task Creation
		currentDateTime = zoho.currenttime;
		reminderTime = currentDateTime.addMinutes(2);
		taskMap = Map();
		taskMap.put("What_Id",lead_id);
		taskMap.put("$se_module","Leads");
		taskMap.put("Owner",owner_id);
		taskMap.put("Subject","Missed Call Follow-up");
		taskMap.put("Status","Not Started");
		taskMap.put("Priority","High");
		taskMap.put("Due_Date",zoho.currentdate);
		taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=POPUP;TRIGGER=DATE-TIME:" + reminderTime.toString("yyyy-MM-dd'T'HH:mm:ssXXX")});
		createdTask = zoho.crm.createRecord("Tasks",taskMap,{"trigger":{"workflow"}});
		info createdTask;
	}
	else
	{
		leadMap = Map();
		leadMap.put("Last_Name","Missed Call - " + call_number);
		leadMap.put("Mobile",call_number);
		leadMap.put("Lead_Status","Fresh Lead");
		leadMap.put("Lead_Source","IVR Missed Call");
		leadMap.put("Owner","991021000000455001");
		create_lead = zoho.crm.createRecord("Leads",leadMap);
		info "else";

		if(create_lead != null && create_lead.containsKey("id"))
		{
			lead_id = create_lead.get("id");
			lead_data = zoho.crm.getRecordById("Leads",lead_id);
			owner_id = lead_data.get("Owner").get("id");
			info zoho.crm.updateRecord("Calls",id,{"$se_module":"Leads","What_Id":lead_id});
			info "2if";
			
			// Task Creation
			currentDateTime = zoho.currenttime;
			reminderTime = currentDateTime.addMinutes(2);
			taskMap = Map();
			taskMap.put("What_Id",lead_id);
			taskMap.put("$se_module","Leads");
			taskMap.put("Owner",owner_id);
			taskMap.put("Subject","Missed Call Follow-up");
			taskMap.put("Status","Not Started");
			taskMap.put("Priority","High");
			taskMap.put("Due_Date",zoho.currentdate);
			taskMap.put("Remind_At",{"ALARM":"FREQ=NONE;ACTION=POPUP;TRIGGER=DATE-TIME:" + reminderTime.toString("yyyy-MM-dd'T'HH:mm:ssXXX")});
			createdTask = zoho.crm.createRecord("Tasks",taskMap,{"trigger":{"workflow"}});
			info createdTask;
		}
	}
}
```

---

# Types of Trigger :

```javascript
const triggerList = ["approval", "workflow", "blueprint", "pathfinder", "orchestration"];
```

## Create / Update Records

```javascript

// Create Record
mp = {"Name":"Joe","Phone":"+1 678 XXX XXXX","Email":"joe@academy.com"};
response = zoho.crm.createRecord("Leads",mp,{"trigger":{"workflow"}});

// Update Record
mp = {"Email":"joe@academy.com"};
response = zoho.crm.updateRecord("Leads",mp,{"trigger":{"workflow"}});

```

## Assignment Rules

```javascript

response = zoho.crm.createRecord("Leads", mp, {"trigger":{"lar_id":"4409363000012741244"}});
// OR
response = zoho.crm.createRecord("Leads", mp, {"trigger":{"workflow","lar_id":"xxxxxxxxxxxxxxxxxxx"}});

```

## Blueprint

```javascript

response = zoho.crm.createRecord("Leads", mp, {"trigger":{"blueprint"}});
// OR
response = zoho.crm.createRecord("Leads", mp, {"trigger":{"workflow","blueprint","lar_id":"xxxxxxxxxxxxxxxxxxx"}});

```

---

# Create custom Signal :
Note : It only works in Lead, Contact & Deal Module.

```javascript

// lead_id = "719220000021334519";

namespace = "notifyreenquiry_notifyreenquiry";
subject = "Re-Enquiry Created - Existing Lead";
message = "Re-Enquiry has been created for existing lead. Lead ID: " + lead_id;
// 	email = lead_data.get("Email"); // optional

signalMap = Map();
signalMap.put("signal_namespace",namespace);
signalMap.put("id",lead_id);
signalMap.put("subject",subject);
signalMap.put("message",message);
result = zoho.crm.raiseSignal(signalMap);
info result;

```

---

# Restrict User to modify field using Validation Rule :

```javascript

entityMap = crmAPIRequest.toMap().get("record");
new_due_date = entityMap.get("Due_Date");
record_id = entityMap.get("id");
if(record_id != null)
{
	existing_record = zoho.crm.getRecordById("Tasks",record_id);
	old_due_date = existing_record.get("Due_Date");
	if(new_due_date != old_due_date)
	{
		response = Map();
		response.put("status","error");
		response.put("message","You are not allowed to edit the Due Date.");
		return response;
	}
	else
	{
		response = Map();
		response.put("status","success");
	}
}
else
{
	response = Map();
	response.put("status","success");
}
return response;

```
---

# Search using COQL :
Note : This function is searching if "1234567890" number contains("like").

```javascript

queryMap = Map();
queryMap.put("select_query","select Mobile from Contacts where Mobile like '%1234567890%' and Mobile is not null limit 1000");
response = invokeurl
[
	url :"https://www.zohoapis.in/crm/v8/coql"
	type :POST
	parameters:queryMap.toString()
	connection:"zohocoql"
];
info response;

```
---

# Send WhatsApp message through WA template :

```javascript
// https://survey.zohopublic.in/zs/JMDw5P
// https://survey.zohopublic.in/zs/JMDw5P?fromservice=ZCRM&
// https://survey.zohopublic.in/zs/JMDw5P?fromservice=ZCRM&zs_custommodule2=

res = zoho.crm.getRecordById("IVR_Calls",id);
Mobile_No = res.get("Caller_ID");
info Mobile_No;
if(Mobile_No != null)
{
	// Prepare WhatsApp API request details
	last_ten = Mobile_No.subString(Mobile_No.length() - 10);
	// info last_ten;

	toNumber = "+91" + last_ten;
	URL = "https://graph.facebook.com/v18.0/101496606391578/messages";

	// API headers (content type + authentication token)
	header = Map();
	header.put("Content-Type","application/json");
	header.put("Authorization","api key ex-> Bearer EAACJxqiRZBPABOxsevDhr7tj");

	// Create request body for WhatsApp template message
	body = Map();
	body.put("messaging_product","whatsapp");
	body.put("recipient_type","individual");
	body.put("to",toNumber);
	body.put("type","template");

	// Define WhatsApp template details
	template = Map();
	template.put("name","thanks_for_calling");
	template.put("language",{"code":"en"});

	// Add template components (header image, body text, CTA button)
	components = List();
	imgVariable = {"type":"header","parameters":{{"type":"image","image":{"link":"https://www.msh.org.in/img/logo1.png"}}}};
	components.add(imgVariable);
	bodyVariable = {"type":"body"};
	components.add(bodyVariable);

	//CTA button
	buttonVariable = {"type":"button","sub_type":"url","index":"0","parameters":{{"type":"text","text":id.toString()}}};
	components.add(buttonVariable);
	template.put("components",components);
	body.put("template",template);
	// info header;
	// info body;

	Response = invokeurl
	[
		url :URL
		type :POST
		parameters:body
		headers:header
		detailed:true
	];
	info Response;

	updateMp = {"What_s_App_Response":Response};
	update_record = zoho.crm.updateRecord("IVR_Calls",id,updateMp);
}
else
{
	info "mobile prob!";
}

```
---
# Search using COQL :
Note: This function checks how many records exist for the given mobile number in *IVR_Calls* for the current day.

```javascript
	dateStr = zoho.currentdate.toString("yyyy-MM-dd");
	startOfDay = dateStr + "T00:00:00+05:30";
	endOfDay = dateStr + "T23:59:59+05:30";

	queryMap = Map();
	queryMap.put("select_query","select id from IVR_Calls where Caller_ID = '" + Mobile_No + "' and Created_Time between '" + startOfDay + "' and '" + endOfDay + "'");

	searchRes = invokeurl
	[
		url :"https://www.zohoapis.in/crm/v8/coql"
		type :POST
		parameters:queryMap.toString()
		connection:"zohocoql"
	];
	// 	info searchRes;

	records = searchRes.get("data");
	count = ifnull(records.size(),0);

```
---

# Send Mail using Deluge with Zoho Sign Template :

```javascript

data = zoho.crm.getRecordById("Deals",id);
contId = data.get("Contact_Name").get("id");
contData = zoho.crm.getRecordById("Contacts",contId);

// Form input values
recipientName = data.get("Deal_Name");
recipientEmail = contData.get("Email");

// API URL (your template)
url = " https://sign.zoho.com/api/v1/templates/520214000000047059/createdocument";

// Construct request body
dataMap = Map();
actionMap = Map();
actionMap.put("recipient_name", recipientName);
actionMap.put("recipient_email", recipientEmail);
actionMap.put("action_id", "520214000000047080");
actionMap.put("action_type", "SIGN");
actionMap.put("signing_order", 1);
actionMap.put("role", "test");
actionMap.put("verify_recipient", false);
actionMap.put("private_notes", "");
actionsList = List();
actionsList.add(actionMap);

todayDate = zoho.currentdate.toString("dd MMMM yyyy");
fieldDateData = Map();
fieldDateData.put("Date", todayDate);

fieldData = Map();
fieldData.put("field_text_data", Map());
fieldData.put("field_boolean_data", Map());
fieldData.put("field_date_data", fieldDateData);
fieldData.put("field_radio_data", Map());
fieldData.put("field_checkboxgroup_data", Map());

// Template map
templateMap = Map();
templateMap.put("field_data", fieldData);
templateMap.put("notes", "");
templateMap.put("actions", actionsList);

// Final request map
dataMap.put("templates", templateMap);

// API call
response = invokeurl
[
    url : url
    type : POST
    parameters: dataMap.toString()
    connection: "zoho_sign_connection"
];
info response;

// Save Document ID for tracking
if(response.get("requests") != null)
{
    docId = response.get("requests").get("request_id");
    info docId;
}

```

---

# Send mail using deluge with Mail Merge Template (Basic):

```javascript
recordId = "1833590000049735036";

dealData = zoho.crm.getRecordById("Deals", recordId);
deal_owner_id = dealData.get("Owner").get("id");

deal_owner_mail = zoho.crm.getRecordById("users", deal_owner_id).get("users").getJson("email");

contId = dealData.get("Contact_Name").get("id");

contMail = zoho.crm.getRecordById("Contacts", contId).get("Email");

mailMergeMap = Map();

// Template
templateMap = Map();
templateMap.put("name", "Proposal Template");
mailMergeMap.put("mail_merge_template", templateMap);

// From address
fromMap = Map();
fromMap.put("type", "email");
fromMap.put("value", deal_owner_mail);
mailMergeMap.put("from_address", fromMap);

// To address list
toList = List();

to1 = Map();
to1.put("type", "email");
to1.put("value", contMail);

toList.add(to1);

mailMergeMap.put("to_address", toList);

// Subject
mailMergeMap.put("subject", "Internet Moguls Proposal");

// Message body
mailMergeMap.put("message", "");

// Wrap in main list
mailMergeList = List();
mailMergeList.add(mailMergeMap);

// Final request body
requestBody = Map();
requestBody.put("send_mail_merge", mailMergeList);

// API Call
response = invokeurl
[
    url :"https://www.zohoapis.com/crm/v8/Deals/" + recordId + "/actions/send_mail_merge"
    type :POST
    parameters: requestBody.toString()
    connection:"zohocrm"
];

info response;
```

---

# Send mail using deluge with Mail Merge Template (Advance):

```javascript
recordId = "1234567890";

dealData = zoho.crm.getRecordById("Deals", recordId);
deal_owner_id = dealData.get("Owner").get("id");

deal_owner_mail = zoho.crm.getRecordById("users", deal_owner_id).get("users").getJson("email");

contId = dealData.get("Contact_Name").get("id");

contMail = zoho.crm.getRecordById("Contacts", contId).get("Email");

templateName = "mailmergename";
mailMergeMap = Map();

// Template
templateMap = Map();
templateMap.put("name", templateName);
mailMergeMap.put("mail_merge_template", templateMap);

// From address
fromMap = Map();
fromMap.put("type", "email");
fromMap.put("value", deal_owner_mail);
mailMergeMap.put("from_address", fromMap);

// To address list
toList = List();

to1 = Map();
to1.put("type", "email");
to1.put("value", contMail);

toList.add(to1);

//..........//
to2 = Map();
to2.put("type", "email");
to2.put("value", "test2@gmail.com");

toList.add(to2);
//..........//

mailMergeMap.put("to_address", toList);

// Subject
mailMergeMap.put("subject", "Internet Moguls Proposal");

//..........//
// CC emails
ccList = List();
cc = Map();
cc.put("type", "email");
cc.put("value", "brie.c@gmail.com");
ccList.add(cc);

mailMergeMap.put("cc_email", ccList);

// BCC emails
bccList = List();
bcc = Map();
bcc.put("type", "email");
bcc.put("value", "ceo.zylker@gmail.com");
bccList.add(bcc);

mailMergeMap.put("bcc_email", bccList);

// Attachment settings
mailMergeMap.put("type", "attachment");
mailMergeMap.put("attachment_name", "testdocument");
//..........//

// Message body
mailMergeMap.put("message", "");

// Wrap in main list
mailMergeList = List();
mailMergeList.add(mailMergeMap);

// Final request body
requestBody = Map();
requestBody.put("send_mail_merge", mailMergeList);

// API Call
response = invokeurl
[
    url :"https://www.zohoapis.com/crm/v8/Deals/" + recordId + "/actions/send_mail_merge"
    type :POST
    parameters: requestBody.toString()
    connection:"zohocrm"
];

info response;
```

---

# Auto lead conversion using webhook :

```javascript
string standalone.audit_survey(map crmAPIRequest)
{
    info crmAPIRequest;
 
    bodyMap = crmAPIRequest.get("body");
 
    if(bodyMap == null)
    {
        return "Error: Body missing";
    }
 
    responseId = bodyMap.get("resp_id");
 
    if(responseId == null)
    {
        return "Error: resp_id missing";
    }
 
    // Search Lead
    criteria = "(Response_Id:equals:" + responseId + ")";
    leadSearch = zoho.crm.searchRecords("Leads", criteria);
 
    if(leadSearch == null || leadSearch.size() == 0)
    {
        return "Error: No Lead found";
    }
 
    leadData = leadSearch.get(0);
    leadId = leadData.get("id");
 
    // Prevent duplicate conversion
    if(leadData.get("Converted_Contact_Id") != null)
    {
        return "Already converted";
    }
 
	// Convert Lead
	convertMap = Map();
	convertMap.put("overwrite", true);
	
	// Get Lead Name
	leadName = leadData.get("Full_Name");
	if(leadName == null)
	{
		leadName = leadData.get("Last_Name"); // fallback
	}
	
	// Deal creation
	dealMap = Map();
	dealMap.put("Deal_Name", leadName);   // Deal = Lead Name
	dealMap.put("Stage", "Qualification");
	
	convertMap.put("Deals", dealMap);
	
	// Convert
	convertResp = zoho.crm.convertLead(leadId.toLong(), convertMap);
	return "Success: " + convertResp.toString();

}
```

---

# Record Delete Method :

```javascript
deleteRecordMap = {"module":"Leads","id":LeadID};
zoho.crm.invokeConnector("crm.delete",deleteRecordMap);
```

---

# Create Lead from IndiaMart using webhook :

```javascript
newMap = crmAPIRequest.get("body").get("RESPONSE").toMap();
queryId = newMap.get("UNIQUE_QUERY_ID");

get_response = zoho.crm.searchRecords("Leads","(indiamartleads__IM_QUERY_ID:equals:" + queryId + ")").toString();

if(get_response == "")
{
	enqAddress = newMap.get("SENDER_ADDRESS");
	mob = newMap.get("SENDER_MOBILE");
	phone = newMap.get("SENDER_MOBILE_ALT");

	if(phone is not null)
	{
		phone = phone.getAlphaNumeric();
	}

	company = newMap.get("SENDER_COMPANY");
	sender_name = newMap.get("SENDER_NAME");
	new_subject = newMap.get("RECEIVER_MOBILE");
	date_time = newMap.get("QUERY_TIME");
	sender_email = newMap.get("SENDER_EMAIL");
	msg = ifnull(newMap.get("QUERY_MESSAGE"),"novalue");

	if(msg != "novalue")
	{
		msg = msg.replaceAll("<br>",".");
		msg = msg.replaceAll("\n","");
		msg = msg.replaceAll("\r","");
	}

	state = newMap.get("SENDER_STATE");
	enq_city = newMap.get("SENDER_CITY");
	product_name = newMap.get("QUERY_PRODUCT_NAME");
	country_iso = newMap.get("SENDER_COUNTRY_ISO");
	im_query_type = newMap.get("QUERY_TYPE");

	if(im_query_type == "W")
	{
		fin_query_type = "Direct Enquiry";
	}
	if(im_query_type == "P")
	{
		fin_query_type = "Call Enquiries";
	}
	if(im_query_type == "B")
	{
		fin_query_type = "Purchased Buy Leads";
	}

	myFieldMap = Map:String();
	myFieldMap.put("Last_Name",sender_name);
	myFieldMap.put("Mobile",mob);
	myFieldMap.put("Phone",phone);
	myFieldMap.put("Email",sender_email);
	myFieldMap.put("Country",country_iso);
	myFieldMap.put("State",state);
	myFieldMap.put("Description",msg);
	myFieldMap.put("Lead_Source","IndiaMart");
	myFieldMap.put("Company",company);
	myFieldMap.put("City",enq_city);
	myFieldMap.put("indiamartleads__IM_QUERY_ID",queryId);
	myFieldMap.put("indiamartleads__IM_SUBJECT",new_subject);
	myFieldMap.put("indiamartleads__IM_ENQUIRY_TIME",date_time);
	myFieldMap.put("indiamartleads__IM_PRODUCT",product_name);
	myFieldMap.put("indiamartleads__IM_Query_TYPE",fin_query_type);
	myFieldMap.put("Street",enqAddress);
	response = zoho.crm.upsert("Leads",myFieldMap,{"trigger":{"workflow"}});

	//response = zoho.crm.createRecord("Leads",myFieldMap,{"trigger":{"workflow","blueprint"}});
	//response = zoho.crm.upsert("Leads",myFieldMap,{"duplicate_check_fields":{"indiamartleads__IM_QUERY_ID"},"trigger":{"workflow"}});
}

return "200";
```
---

# Update Subform using Deluge ( Part - 1 ) :

```javascript
// get parent record
parentRec = zoho.crm.getRecordById("Module_Name", recordId);

// get subform
subform = parentRec.get("Subform_API_Name");

// prepare subform update
subformList = List();

rowMap = Map();
rowMap.put("id", subformRowId);   // mandatory (subform row id)

// update fields (add as needed)
rowMap.put("Field_API_1", value1);
rowMap.put("Field_API_2", value2);

subformList.add(rowMap);

// update parent
updateMap = Map();
updateMap.put("Subform_API_Name", subformList);

resp = zoho.crm.updateRecord("Module_Name", recordId, updateMap);
info resp;
```

---

# Update Specific Subform Row using Deluge ( Part - 2 ) :

```javascript
recordId = "Parent_Record_ID";
targetSubformRowId = "Subform_Row_ID";

value1 = "New Value 1";
value2 = "New Value 2";

// get parent record
parentRec = zoho.crm.getRecordById("Module_Name", recordId);

// get subform
subform = parentRec.get("Subform_API_Name");

subformList = List();

for each row in subform
{
	rowMap = Map();
	rowId = row.get("id");

	// Always preserve the row
	rowMap.put("id", rowId);

	// Update only the target row
	if(targetSubformRowId == rowId)
	{
		rowMap.put("Field_API_1", value1);
		rowMap.put("Field_API_2", value2);
	}

	subformList.add(rowMap);
}

// update parent
updateMap = Map();
updateMap.put("Subform_API_Name", subformList);

resp = zoho.crm.updateRecord("Module_Name", recordId, updateMap);
info resp;
```
---

# Generate Dynamic Invoice Number :

```javascript
id = "1234567890";

inv_data = zoho.crm.getRecordById("Invoices",id);
inv_type = inv_data.get("Invoice_Type");

// get fy
curr_date = zoho.currentdate;
year_val = year(curr_date);
month_val = month(curr_date);

if(month_val >= 4)
{
	start_year = year_val;
	end_year = (year_val + 1).toString().right(2);
}
else
{
	start_year = year_val - 1;
	end_year = year_val.toString().right(2);
}

fy = start_year + "-" + end_year;
info fy;

query = "select Invoice_No from Invoices where Invoice_Type = '" + inv_type + "' and Invoice_No like '%/" + fy + "/%' order by Created_Time desc limit 1";

queryMap = Map();
queryMap.put("select_query",query);

response = invokeurl
[
	url :"https://www.zohoapis.com/crm/v8/coql"
	type :POST
	parameters:queryMap.toString()
	connection:"zohocoql"
];
info response;

prefix = "";

if(inv_type == "IELTS & Visa Fees")
{
	prefix = "GLBD/" + fy;
}
else if(inv_type == "IELTS & Visa Fees Chandigarh")
{
	prefix = "GLB-CHD/" + fy;
}
else if(inv_type == "Commission from India")
{
	prefix = "GLBC/" + fy;
}
else if(inv_type == "Commission from Foreign University")
{
	prefix = "GLB/" + fy;
}

if(response == "")
{
	fullstr = prefix + "/1";
}
else
{
	data = response.get("data");
	inv_no = data.get(0).get("Invoice_No");
	info inv_no;
	parts = inv_no.toList("/");
	last_num = parts.get(2);
	info last_num;
	next_num = last_num.toLong() + 1;
	fullstr = prefix + "/" + next_num;
}
info fullstr;

up_inv = zoho.crm.updateRecord("Invoices",id,{"Invoice_No":fullstr});
info up_inv;
```

---

# Prevent Duplicate Records & Escape XML Special Characters :

Prevent Duplicate Records
```javascript

processedIds = List();

for each record in records
{
	recordId = record.get("id");

	if(processedIds.contains(recordId))
	{
		continue;
	}

	processedIds.add(recordId);

}
```
Escape XML Special Characters

```javascript
// Raw text
value = "do&dont";

// Escape XML characters
value = value.replaceAll("&","&amp;");
value = value.replaceAll("<","&lt;");
value = value.replaceAll(">","&gt;");
value = value.replaceAll("\"","&quot;");
value = value.replaceAll("'","&apos;");
value = value.replaceAll("/","&#x2F;");
value = value.replaceAll("`","&#x60;");

```
Generic Reusable XML Escape Loop
```javascript

// Store raw values
rawFields = Map();
rawFields.put("name","Government & Defense");
rawFields.put("company","A < B Pvt. Ltd.");
rawFields.put("owner","John's Team");

// Escape all fields dynamically
escapedFields = Map();

for each key in rawFields.keys()
{
	val = ifnull(rawFields.get(key),"---").toString();

	value = value.replaceAll("&","&amp;");
	value = value.replaceAll("<","&lt;");
	value = value.replaceAll(">","&gt;");
	value = value.replaceAll("\"","&quot;");
	value = value.replaceAll("'","&apos;");
	value = value.replaceAll("/","&#x2F;");
	value = value.replaceAll("`","&#x60;");

	escapedFields.put(key,val);
}

// Get escaped values
safeName = escapedFields.get("name");
safeCompany = escapedFields.get("company");
safeOwner = escapedFields.get("owner");
```
---

# Send Leads to Meta via Conversions API (CAPI) :

```javascript
Note: Hashes email & phone (SHA-256) before sending.

// Fetch Lead
lead = zoho.crm.getRecordById("Leads",id);

// Lead Details
firstName = lead.get("First_Name");
lastName = lead.get("Last_Name");
email = lead.get("Email");
mobile = lead.get("Mobile");

// UTM Fields
utmCampaign = lead.get("UTM_Campaign");
utmMedium = lead.get("UTM_Medium");
utmTerm = lead.get("UTM_Term");
utmAdvert = lead.get("UTM_Advert");

// Hash Email & Mobile
hashedFn = "";
hashedLn = "";
hashedEmail = "";
hashedPhone = "";

if(firstName != null && firstName != "")
{
	hashedFn = zoho.encryption.sha256(firstName.toLowerCase());
}

if(lastName != null && lastName != "")
{
	hashedLn = zoho.encryption.sha256(lastName.toLowerCase());
}

if(email != null && email != "")
{
	hashedEmail = zoho.encryption.sha256(email.toLowerCase());
}

if(mobile != null && mobile != "")
{
	// cleanPhone = mobile.replaceAll("[^0-9]", "");
	hashedPhone = zoho.encryption.sha256(mobile);
}

// Event Timestamp (Unix)
eventTime = zoho.currenttime.toLong() / 1000;

// Build User Data
userData = Map();

userData.put("lead_id",id);

if(hashedFn != "")
{
	fnList = List();
	fnList.add(hashedFn);
	userData.put("fn",fnList);
}

if(hashedLn != "")
{
	lnList = List();
	lnList.add(hashedLn);
	userData.put("ln",lnList);
}

if(hashedEmail != "")
{
	emList = List();
	emList.add(hashedEmail);
	userData.put("em",emList);
}

if(hashedPhone != "")
{
	phList = List();
	phList.add(hashedPhone);
	userData.put("ph",phList);
}

// Custom Data
customData = Map();
customData.put("event_source","crm");
customData.put("lead_event_source","Zoho CRM");
customData.put("utm_campaign",utmCampaign);
customData.put("utm_medium",utmMedium);
customData.put("utm_term",utmTerm);
customData.put("utm_advert",utmAdvert);

// Event Object
eventObj = Map();
eventObj.put("action_source","system_generated");
eventObj.put("event_name","Lead");
eventObj.put("event_time",eventTime);
eventObj.put("user_data",userData);
eventObj.put("custom_data",customData);

// Data Array
dataList = List();
dataList.add(eventObj);

// Meta Credentials
pixelId = "727111683624460";
accessToken = "EAAU9d1ho38wB";

// Payload
payload = Map();
payload.put("data",dataList);
payload.put("access_token",accessToken);

// API Call
response = invokeurl
[
	url :"https://graph.facebook.com/v18.0/" + pixelId + "/events"
	type :POST
	parameters:payload.toString()
	headers:{"Content-Type":"application/json"}
];

info response;

```
---

# Create Subform using Deluge ( Part - 1 ) :

```javascript
// prepare subform
subformList = List();

rowMap = Map();

// add fields (as needed)
rowMap.put("Field_API_1", value1);
rowMap.put("Field_API_2", value2);

subformList.add(rowMap);

// add to parent map
parentMap = Map();
parentMap.put("Subform_API_Name", subformList);

// create parent record
resp = zoho.crm.createRecord("Module_Name", parentMap);
info resp;
```

Json Format -- for Creating Single Subform Row --
```json
{
  "Subform_API_Name": [
    {
      "Field_API_1": "value1",
      "Field_API_2": "value2"
    }
  ]
}
```
---

# Create Multiple Subform Rows using Deluge ( Part - 2 ) :

```javascript
// prepare subform
subformList = List();

for each item in sourceList
{
	rowMap = Map();

	// map fields (as needed)
	rowMap.put("Field_API_1", item.get("Key_1"));
	rowMap.put("Field_API_2", item.get("Key_2"));

	subformList.add(rowMap);
}

// add to parent map
parentMap = Map();
parentMap.put("Subform_API_Name", subformList);

// create parent record
resp = zoho.crm.createRecord("Module_Name", parentMap);
info resp;
```

Json Format -- for Creating Multiple Subform Rows --
```json
{
  "Subform_API_Name": [
    {
      "Field_API_1": "value1",
      "Field_API_2": "value2"
    },
    {
      "Field_API_1": "value3",
      "Field_API_2": "value4"
    },
    {
      "Field_API_1": "value5",
      "Field_API_2": "value6"
    }
  ]
}
```
---

# Auto Generate Monthly Collection Records from Deals :
Note - Scheduler-based function that runs automatically on the last day of every month. It retrieves active Deals using COQL for the current billing month, calculates prorated rental charges for the first and last billing months using the Closing Date and Agreement End Date, applies the full monthly rental amount for intermediate months, and automatically creates Collection records with the appropriate Invoice Date and Rental Amount.

```javascript
today = zoho.currentdate;
firstDay = today.toString("yyyy-MM") + "-01";
lastDay = today.eomonth(0).toString("yyyy-MM-dd");
currentMonth = today.getMonth();
currentYear = today.getYear();
daysInMonth = today.eomonth(0).getDay();

queryMap = Map();
queryMap.put("select_query","select id, Deal_Name, Agreement_End_Date, Amount, Closing_Date, Genset, Account_Name from Deals where Closing_Date <= '" + lastDay + "' and Agreement_End_Date >= '" + firstDay + "' limit 0,2000");

response = invokeurl
[
	url :"https://www.zohoapis.com/crm/v8/coql"
	type :POST
	parameters:queryMap.toString()
	connection:"zohocoql"
];

info response;

deal_data = response.get("data");

for each deal in deal_data
{
	amount = ifnull(deal.get("Amount"),0).toDecimal();
	closingDate = deal.get("Closing_Date").toDate();
	agreementEndDate = deal.get("Agreement_End_Date").toDate();

	isFirst = (closingDate.getMonth() == currentMonth && closingDate.getYear() == currentYear);
	isLast = (agreementEndDate.getMonth() == currentMonth && agreementEndDate.getYear() == currentYear);

	invoiceAmount = amount;
	invoiceDate = firstDay;

	if(isFirst && isLast)
	{
		billableDays = agreementEndDate.getDay() - closingDate.getDay() + 1;
		invoiceAmount = ((amount / daysInMonth) * billableDays).round(2);
		invoiceDate = closingDate;
	}

	else if(isFirst)
	{
		if(closingDate.getDay() == 1)
		{
			invoiceAmount = amount;
		}
		else
		{
			billableDays = daysInMonth - closingDate.getDay() + 1;
			invoiceAmount = ((amount / daysInMonth) * billableDays).round(2);
		}
		invoiceDate = closingDate;
	}

	else if(isLast)
	{
		if(agreementEndDate.getDay() == daysInMonth)
		{
			invoiceAmount = amount;
		}
		else
		{
			invoiceAmount = ((amount / daysInMonth) * agreementEndDate.getDay()).round(2);
		}
		invoiceDate = agreementEndDate;
	}
	
	else
	{
		invoiceAmount = amount;
		invoiceDate = firstDay;
	}

	siteId = if(deal.get("Account_Name") != null, deal.get("Account_Name").get("id"), "");
	gensetId = if(deal.get("Genset") != null, deal.get("Genset").get("id"), "");

	mp = Map();
	mp.put("Name",deal.get("Deal_Name"));
	mp.put("Rental_Pipeline",deal.get("id"));
	mp.put("Rental_Amount",invoiceAmount);
	mp.put("Invoice_Date",invoiceDate);
	mp.put("Site",siteId);
	mp.put("Installed_Genset",gensetId);

	create = zoho.crm.createRecord("Collections",mp);
	info create;
	
// 	break;
}
```
---

# Calculate and Update Monthly Net Total on Rental Collection Record :
Note - Calculates the Monthly Net Total for a given month by summing all Rental Amounts and Monthly Expenses from Collections records with the same month/year, then updates the corresponding Rental record with the calculated net value (Rental Amount Total − Monthly Expenses Total).

```javascript
// Get current Collections record
record = zoho.crm.getRecordById("Collections",id);
nameValue = ifnull(record.get("Name"),"");

// Find month
months = {"January","February","March","April","May","June","July","August","September","October","November","December"};
monthFound = "";

for each  month in months
{
	if(nameValue.contains(month))
	{
		monthFound = month;
		break;
	}
}

// Find year
yearFound = "";
yearIndex = nameValue.indexOf("20");

if(yearIndex != -1)
{
	yearFound = nameValue.subString(yearIndex,yearIndex + 4);
}

searchText = monthFound + " " + yearFound;

if(monthFound == "" || yearFound == "")
{
	info "Could not determine Month/Year from Name: " + nameValue;
	return;
}

info "Searching for: " + searchText;

totalRentalAmount = 0.0;
totalMonthlyExpenses = 0.0;
rentalRecordId = "";

// Fetch all records for the same Month & Year
query = "select id, Collection_Type, Rental_Amount, Monthly_Expenses from Collections where Name like '%" + searchText + "%'";
queryMap = Map();
queryMap.put("select_query",query);

response = invokeurl
[
	url :"https://www.zohoapis.com/crm/v8/coql"
	type :POST
	parameters:queryMap.toString()
	connection:"zohocoql"
];

if(response.containsKey("data"))
{
	records = response.get("data");
	for each  rec in records
	{
		totalRentalAmount = totalRentalAmount + ifnull(rec.get("Rental_Amount"),0);
		totalMonthlyExpenses = totalMonthlyExpenses + ifnull(rec.get("Monthly_Expenses"),0);
		if(rec.get("Collection_Type") == "Rental")
		{
			rentalRecordId = rec.get("id");
		}
	}
}

monthlyNetTotal = totalRentalAmount - totalMonthlyExpenses;
info "Total Rental Amount : " + totalRentalAmount;
info "Total Monthly Expenses : " + totalMonthlyExpenses;
info "Monthly Net Total : " + monthlyNetTotal;

if(rentalRecordId != "")
{
	updateMap = Map();
	updateMap.put("Monthly_Net_Total",monthlyNetTotal);
	updateResponse = zoho.crm.updateRecord("Collections",rentalRecordId,updateMap);
	info "Rental Record ID : " + rentalRecordId;
	info updateResponse;
}
else
{
	info "No Rental record found for " + searchText;
}
```
---

# Monthly Collection Auto-Creation Scheduler :
Note - Scheduler function that runs on the 1st of every month to reset eligible Collection records to "Pending", generate Rental Collection records for all active Deals in the current billing month, calculate full or prorated rental amounts based on the agreement start/end dates, and create new Collection records with the appropriate invoice date, rental amount, and related Deal, Site, and Genset information.

```javascript
// Collections Status Update using CVID + Pages Method
cvid = "7342033000003269006";
module = "Collections";
pages = {1,2,3,4,5,6,7,8,9,10};
for each  pg in pages
{
	collection_data = zoho.crm.getRecords(module,1,200,{"cvid":cvid});
	// always page 1
	if(collection_data.size() > 0)
	{
		for each  rec in collection_data
		{
			rec_id = rec.get("id");
			updateMap = Map();
			updateMap.put("Status","Pending");
			res = zoho.crm.updateRecord(module,rec_id,updateMap);
			info res;
			// 			break;
		}
	}
	else
	{
		break;
	}
	info "Batch : " + pg + " processed";
	// 	break;
}
info "done";

// create collections
today = zoho.currentdate;

// Bill CURRENT month in advance
billingDate = today;
firstDay = billingDate.toString("yyyy-MM") + "-01";
lastDay = billingDate.eomonth(0).toString("yyyy-MM-dd");
currentMonth = billingDate.getMonth();
currentYear = billingDate.getYear();
daysInMonth = billingDate.eomonth(0).getDay();

queryMap = Map();

// 	"select id, Deal_Name, Agreement_End_Date, Amount, Closing_Date, Genset, Account_Name from Deals where Closing_Date <= '" + lastDay + "' and Agreement_End_Date >= '" + firstDay + "' limit 0,2000"

select_query = "select id, Deal_Name, Agreement_End_Date, Amount, Closing_Date, Genset, Account_Name from Deals where ((Closing_Date <= '" + lastDay + "' and Agreement_End_Date >= '" + firstDay + "') and Stage != 'Closed Lost') limit 0,2000";

queryMap.put("select_query",select_query);

response = invokeurl
[
	url :"https://www.zohoapis.com/crm/v8/coql"
	type :POST
	parameters:queryMap.toString()
	connection:"zohocoql"
];

info response;

deal_data = response.get("data");
for each  deal in deal_data
{
	amount = ifnull(deal.get("Amount"),0).toDecimal();
	closingDate = deal.get("Closing_Date").toDate();
	agreementEndDate = deal.get("Agreement_End_Date").toDate();
	isFirst = closingDate.getMonth() == currentMonth && closingDate.getYear() == currentYear;
	isLast = agreementEndDate.getMonth() == currentMonth && agreementEndDate.getYear() == currentYear;

	invoiceAmount = amount;
	invoiceDate = lastDay;

	if(isFirst && isLast)
	{
		billableDays = agreementEndDate.getDay() - closingDate.getDay() + 1;
		invoiceAmount = (amount / daysInMonth * billableDays).round(2);
		invoiceDate = agreementEndDate;
	}
	else if(isFirst)
	{
		if(closingDate.getDay() == 1)
		{
			invoiceAmount = amount;
		}
		else
		{
			billableDays = daysInMonth - closingDate.getDay() + 1;
			invoiceAmount = (amount / daysInMonth * billableDays).round(2);
		}
		invoiceDate = lastDay;
	}
	else if(isLast)
	{
		if(agreementEndDate.getDay() == daysInMonth)
		{
			invoiceAmount = amount;
		}
		else
		{
			invoiceAmount = (amount / daysInMonth * agreementEndDate.getDay()).round(2);
		}
		invoiceDate = agreementEndDate;
	}
	else
	{
		invoiceAmount = amount;
		invoiceDate = lastDay;
	}
	siteId = if(deal.get("Account_Name") != null,deal.get("Account_Name").get("id"),"");
	siteName = if(deal.get("Account_Name") != null,deal.get("Account_Name").get("name"),"");
	gensetId = if(deal.get("Genset") != null,deal.get("Genset").get("id"),"");
	startDateText = billingDate.toString("d MMMM yyyy");
	endDateText = billingDate.eomonth(0).toString("d MMMM yyyy");

	// 	collectionName = "(" + startDateText + " - " + endDateText + ") - " + siteName;
	collectionType = "Rental";
	collectionName = "(" + startDateText + " - " + endDateText + ") - " + collectionType + " - " + siteName;

	mp = Map();
	mp.put("Name",collectionName);
	mp.put("Rental_Pipeline",deal.get("id"));
	mp.put("Rental_Amount",invoiceAmount);
	mp.put("Monthly_Net_Total",invoiceAmount);
	mp.put("Invoice_Date",invoiceDate);
	mp.put("Site",siteId);
	mp.put("Installed_Genset",gensetId);
	mp.put("Status","Active");
	mp.put("Collection_Type","Rental");

	create = zoho.crm.createRecord("Collections",mp);
	info create;
	// 	break;
}
```
---

# Update Rental Pipeline (Deal) Name After Transfer :
Note - Standalone function that retrieves the latest Transfer record for a Rental Pipeline (Deal), extracts the uninstallation date, and updates the Deal Name by refreshing the genset rating and appending or replacing the transfer (TR) date while preserving the existing naming convention.

```javascript
rentalId = "1234567890";
dealName = "siteName/branchName/30KVA";
gensetRating = "30KVA";

queryMap = Map();

// Pulls the most recent Transfer record with specified fields
coqlQuery = "select id, Name, Uninstalled_Date from Transfers where Rental_Pipeline = '" + rentalId + "' order by Created_Time desc limit 1";
queryMap.put("select_query",coqlQuery);

response = invokeurl
[
	url :"https://www.zohoapis.com/crm/v8/coql"
	type :POST
	parameters:queryMap.toString()
	connection:"zohocoql"
];
info response;

if(response.isEmpty())
{
	info "empty";
	return "return empty - coql";
}

uninstallationDate = response.getJSON("data").getJSON("Uninstalled_Date").toString("dd MMM yyyy");
info uninstallationDate;

if(dealName.containsIgnoreCase("TR"))
{
	arr = dealName.toList("/");
	newName = arr.get(0) + "/" + arr.get(1) + "/" + if(gensetRating != null && gensetRating != "",gensetRating,arr.get(2)) + "/" + if(uninstallationDate != null && uninstallationDate != "","TR-" + uninstallationDate,arr.get(3));
}
else
{
	arr = dealName.toList("/");
	newName = arr.get(0) + "/" + arr.get(1) + "/" + if(gensetRating != null && gensetRating != "",gensetRating,arr.get(2));

	if(uninstallationDate != null && uninstallationDate != "")
	{
		newName = newName + "/TR-" + uninstallationDate;
	}
}
info newName;

update = zoho.crm.updateRecord("Deals",rentalId,{"Deal_Name":newName});
return update;
```
---

# Restrict Date Changes & Prevent Past Date Entry using Validation Rule :
Note - A Zoho CRM Deluge validation function for the Events module that prevents users from changing the date of an existing event (while allowing time edits) and blocks past dates on new event creation. Returns specific error messages identifying whether the From date, To date, or both were modified.

```javascript
entityMap = crmAPIRequest.toMap().get("record");
new_start_datetime = entityMap.get("Start_DateTime");
new_end_datetime = entityMap.get("End_DateTime");
record_id = entityMap.get("id");

if(record_id != null)
{
    // Edit scenario - prevent date change, allow time change
    existing_record = zoho.crm.getRecordById("Events", record_id);
    old_start_datetime = existing_record.get("Start_DateTime");
    old_end_datetime = existing_record.get("End_DateTime");

    // Extract date portions only (YYYY-MM-DD) by splitting on "T"
    old_start_date = old_start_datetime.toString().toList("T").get(0);
    old_end_date = old_end_datetime.toString().toList("T").get(0);
    new_start_date = new_start_datetime.toString().toList("T").get(0);
    new_end_date = new_end_datetime.toString().toList("T").get(0);

    if(new_start_date != old_start_date && new_end_date != old_end_date)
    {
        response = Map();
        response.put("status", "error");
        response.put("message", "You are not allowed to change the From date and the To date. Only time modifications are permitted.");
        return response;
    }
    else if(new_start_date != old_start_date)
    {
        response = Map();
        response.put("status", "error");
        response.put("message", "You are not allowed to change the From date. Only time modifications are permitted.");
        return response;
    }
    else if(new_end_date != old_end_date)
    {
        response = Map();
        response.put("status", "error");
        response.put("message", "You are not allowed to change the To date. Only time modifications are permitted.");
        return response;
    }
    else
    {
        response = Map();
        response.put("status", "success");
    }
}
else
{
    // New record scenario - prevent past date
    today = zoho.currentdate;
    new_start_date = new_start_datetime.toString().toList("T").get(0).toDate();

    if(new_start_date < today)
    {
        response = Map();
        response.put("status", "error");
        response.put("message", "You are not allowed to set a past date for the event.");
        return response;
    }
    else
    {
        response = Map();
        response.put("status", "success");
    }
}

return response;
```
---

# Restrict Past Date Entry (Beyond Yesterday) on New Event Creation using Validation Rule :
Note - A Zoho CRM Deluge validation function for the Events module that runs only on new event creation (record_id is null). It allows the event's Start date to be set to yesterday or later, but blocks creation if the Start date is earlier than yesterday. Edit scenarios (record_id present) are not validated by this function and always return success.

```javascript
entityMap = crmAPIRequest.toMap().get("record");
new_start_datetime = entityMap.get("Start_DateTime");
new_end_datetime = entityMap.get("End_DateTime");
record_id = entityMap.get("id");

if(record_id == null)
{
	// New record scenario - allow yesterday, block anything earlier
	today = zoho.currentdate;
	yesterday = today.addDay(-1);
	new_start_date = new_start_datetime.toString().toList("T").get(0).toDate();

	if(new_start_date < yesterday)
	{
		response = Map();
		response.put("status", "error");
		response.put("message", "You are not allowed to set a date earlier than yesterday for the event.");
		return response;
	}
	else
	{
		response = Map();
		response.put("status", "success");
	}
}
else
{
	response = Map();
	response.put("status", "success");
}
return response;
```
---

# Create Current Month Travel Expense Report in Zoho Expense :
Note - Automatically creates a Travel Expense report in Zoho Expense for the current calendar month. The script calculates the first and last day of the current month, generates a report name in the “MMMM yyyy Report” format, creates the report through the Zoho Expense API, and logs the newly created expense report ID.

```
// Runs on day 1 of every month (configured in the Schedule setup, not here)

organizationId = "60081835784";
today = zoho.currentdate;

// First day of the current month, regardless of what day "today" actually is
monthStartDate = today.addDay(1 - today.getDay());

// Last day of the current month
monthEndDate = today.eomonth(0);
reportName = monthStartDate.toString("MMMM yyyy") + " Report";

parameters_data = Map();
parameters_data.put("report_name",reportName);
parameters_data.put("purpose","Travel Expense");
parameters_data.put("description","");
parameters_data.put("start_date",monthStartDate.toString("yyyy-MM-dd"));
parameters_data.put("end_date",monthEndDate.toString("yyyy-MM-dd"));

headers_data = Map();
headers_data.put("X-in-zoho-expense-organizationid",organizationId);

response = invokeurl
[
	url :"https://www.zohoapis.in/expense/v1/expensereports"
	type :POST
	headers:headers_data
	parameters:parameters_data.toString()
	connection:"zexpense"
];
info response;

reportId = response.get("expense_report").get("report_id");
info "Created Expense Report ID: " + reportId;
```
---