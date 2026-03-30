# Get data from CRM and Show in redirect HTML Page in ZOHO CREATOR :

OnLoad -

```javascript
// Display Company Logo
image_url = "https://www.abhinav.com/wp-content/uploads/2024/08/Ahinav-new-logo-png.png";
input.plain = "<div style=\"text-align:center;\"><img src=\"" + image_url + "\" style=\"height:50px;\" alt=\"Logo\"></div>";

// Hide DOB Field & Submit Button
hide Date_of_Birth;
input.Add_Notes = "<style>input.zc-live-primary-btn{display:none;}</style>";

```

workflow function -


```javascript
mobile = input.Registered_Mobile;
dob = input.Date_of_Birth;
contact_id = list();
if(mobile.startsWith("+91"))
{
	mobile = mobile.substring(3);
}

// First search: Mobile
critera = "(Mobile:equals:" + mobile + ")";
search_contact = zoho.crm.searchRecords("Contacts",critera);
if(search_contact.size() > 0)
{
	contact_dob = search_contact.get(0).get("Date_of_Birth");
	if(contact_dob != null && contact_dob == dob.toString("yyyy-MM-dd"))
	{
		contact_id = search_contact.get(0).get("id").toLong();
	}
}

page_url = "https://creatorapp.zohopublic.com/sanjay645/application-tracker/page-perma/Show_Deal_And_Process_stage/wa8J2R1JXevb4J7naY7BRpDUkO7wnFPjxsJmwfBxT6Gj0ZUmnwDz9uSuHPjm2xDKZJKCDanx5VT7Og7ss5shyYYTT7WOR7df1nOp" + "?uid=" + contact_id;
openUrl(page_url,"same window");

// + if(search_with_dob.size() > 0,"","&bd=" + isDob);
//  + "&bd=" + isDob;

// mobile = zoho.encryption.base64Encode(mobile);
// openUrl("#Page:Show_Deal_And_Process_stage?Registered_Mobile=" + mobile,"same window");

// detail = https://creatorapp.zohopublic.com/sanjay645/application-tracker/form-perma/Customer_Detail/DxkPY6WrhORFuajTa7UgNSFPwdgukrqAbDSsWHT3Eq5PfXstBZPenmKQXuyZQS6FEVyxGPxaGMy09WEnAS7fvdB36EKms3X0FOnJ

//   page = https://creatorapp.zohopublic.com/sanjay645/application-tracker/page-perma/Show_Deal_And_Process_stage/wa8J2R1JXevb4J7naY7BRpDUkO7wnFPjxsJmwfBxT6Gj0ZUmnwDz9uSuHPjm2xDKZJKCDanx5VT7Og7ss5shyYYTT7WOR7df1nOp

```

Checking If Moile is null or not ( function ) -


```javascript
if(input.Checkbox.isEmpty())
{
	hide Date_of_Birth;
	input.Add_Notes = "<style>input.zc-live-primary-btn{display:none;}</style>";
	enable Registered_Mobile;
	return;
}

mobile = input.Registered_Mobile;
if(mobile == null || mobile.trim() == "")
{
	alert "Mobile number is required to proceed. Please enter it first.";
	input.Checkbox = List();
	enable Registered_Mobile;
	return;
}

if(mobile.startsWith("+91"))
{
	mobile = mobile.substring(3);
}

if(ifNull(mobile,"").matches("^[0-9]{10}$") == false)
{
	alert "Please enter a valid 10-digit mobile number.";
	input.Checkbox = List();
	enable Registered_Mobile;
	return;
}

disable Registered_Mobile;
criteria = "(Mobile:equals:" + mobile + ")";
search_contact = zoho.crm.searchRecords("Contacts",criteria);
if(search_contact.size() == 0)
{
	alert "No account is associated with the provided mobile number.";
	hide Date_of_Birth;
	input.Add_Notes = "<style>input.zc-live-primary-btn{display:none;}</style>";
	input.Checkbox = List();
	input.Registered_Mobile = "";
	enable Registered_Mobile;
	return;
}

contact = search_contact.get(0);
contact_dob = contact.get("Date_of_Birth");
contact_owner = contact.get("Owner").get("name");
if(contact_dob == null)
{
	alert "Date of Birth is missing. Please contact your account manager - " + contact_owner;
	hide Date_of_Birth;
	input.Add_Notes = "<style>input.zc-live-primary-btn{display:none;}</style>";
	input.Checkbox = List();
	input.Registered_Mobile = "";
	enable Registered_Mobile;
}
else
{
	show Date_of_Birth;
	input.Add_Notes = "<style>input.zc-live-primary-btn{}</style>";
}
```

html snippet ( redirect ) -

```javascript
<%{
	// 	uid = zoho.encryption.base64Decode(uid); 
	// 	criteria = "(Mobile:equals:" + uid + ")";
	// 	search_contact = zoho.crm.searchRecords("Contacts",criteria);
	// 	related_deals = if(search_contact.size() > 0,zoho.crm.getRelatedRecords("Deals","Contacts",search_contact.get(0).get("id").toLong()),list());	
	// 	form_url = "#Form:Customer_Detail";

	input.uid = input.uid;
	related_deals = if(input.uid.isNumber(),zoho.crm.getRelatedRecords("Deals","Contacts",input.uid.toLong()),list());

	form_url = "https://creatorapp.zohopublic.com/sanjay645/application-tracker/form-perma/Customer_Detail/DxkPY6WrhORFuajTa7UgNSFPwdgukrqAbDSsWHT3Eq5PfXstBZPenmKQXuyZQS6FEVyxGPxaGMy09WEnAS7fvdB36EKms3X0FOnJ";

	%>

<div style="font-family: 'Segoe UI', Arial, sans-serif; background-color: #f4f7f6; padding: 25px; border-radius: 12px;">
    <h1 style="color: #333; text-align: center; margin-bottom: 25px;">Application Details</h1>
	<div style="margin-bottom: 20px;">
	<!-- BACK TO SEARCH BUTTON -->
    <a href=<%=form_url%> style="text-decoration: none;">
        <button style="padding: 10px 20px; background-color: #6c757d; color: white; border: none; border-radius: 5px; cursor: pointer;">
            ← Back to Search
        </button>
    </a>
</div>

<%

	if(related_deals.size() > 0)
	{
		for each  deal in related_deals
		{
			deal_id = deal.get("id").toLong();
			// related_cases = zoho.crm.getRelatedRecords("Cases","Deals",deal_id);
			related_cases_raw = zoho.crm.getRelatedRecords("Cases","Deals",deal_id);
			sortKeys = List();
			caseMap = Map();
			related_cases = List();
			for each  rec in related_cases_raw
			{
				sortVal = ifnull(rec.get("Subject"),"") + "_" + rec.get("id");
				sortKeys.add(sortVal);
				caseMap.put(sortVal,rec);
			}
			sortKeys.sort(true);
			for each  key in sortKeys
			{
				related_cases.add(caseMap.get(key));
			}

			%>

<div style="background-color: #ffffff; padding: 20px; border-radius: 10px; border-left: 6px solid #20B2E3; box-shadow: 0 4px 6px rgba(0,0,0,0.1); margin-bottom: 30px;">
        <h2 style="margin-top: 0; color: #20B2E3; font-size: 20px; border-bottom: 1px solid #eee; padding-bottom: 10px;">Deal: <%=deal.get("Deal_Name")%></h2>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 12px; font-size: 15px; padding-top: 10px;">
            <div><strong>Deal Stage:</strong> <%=deal.get("Stage")%></div>
            <div><strong>Deal Owner:</strong> <%=ifnull(deal.get("Owner").get("name"),"N/A")%></div>
        </div>

        <div style="display: flex; gap: 20px; margin-top: 25px; flex-wrap: wrap;">
            
            <!-- OPEN COLUMN -->
            <div style="flex: 1; min-width: 300px;">
                <h3 style="color: #28a745; font-size: 16px; border-bottom: 2px solid #28a745; padding-bottom: 5px;">OPEN PROCESSES</h3>
<%
			open_count = 0;
			for each  case_rec in related_cases
			{
				if(case_rec.get("Status") != "Closed")
				{
					open_count = open_count + 1;
					%>
<div style="background-color: #e8f5e9; border: 1px solid #c8e6c9; padding: 12px; border-radius: 6px; margin-top: 10px;">
                    <div style="font-weight: bold; color: #2e7d32;"><%=case_rec.get("Subject")%></div>
                    <div style="font-size: 13px; margin-top: 4px;"><strong>Status:</strong> <%=case_rec.get("Status")%></div>
                    <div style="font-size: 13px;"><strong>Owner:</strong> <%=ifnull(case_rec.get("Owner").get("name"),"N/A")%></div>
                </div>
<%
				}
			}
			if(open_count == 0)
			{
				%>
<div style="color:#999; padding-top:10px; font-style:italic;">No open processes.</div>

<%
			}
			%>
</div>

            <!-- CLOSED COLUMN -->
            <div style="flex: 1; min-width: 300px;">
                <h3 style="color: #dc3545; font-size: 16px; border-bottom: 2px solid #dc3545; padding-bottom: 5px;">CLOSED PROCESSES</h3>
<%
			closed_count = 0;
			for each  case_rec in related_cases
			{
				if(case_rec.get("Status") == "Closed")
				{
					closed_count = closed_count + 1;
					%>
<div style="background-color: #FFF5F5; border: 1px solid #f5c6cb; padding: 12px; border-radius: 6px; margin-top: 10px;">
                    <div style="font-weight: bold; color: #721c24;"><%=case_rec.get("Subject")%></div>
                    <div style="font-size: 13px; margin-top: 4px;"><strong>Status:</strong> <%=case_rec.get("Status")%></div>
                    <div style="font-size: 13px;"><strong>Owner:</strong> <%=ifnull(case_rec.get("Owner").get("name"),"N/A")%></div>
                </div>
<%
				}
			}
			if(closed_count == 0)
			{
				%>
<div style="color:#999; padding-top:10px; font-style:italic;">No closed processes.</div>

<%
			}

			%>
</div>

        </div>
    </div>

<%
		}
	}
	else
	{
		%>
<div style="text-align: center; padding: 40px; background-color: #ffe5e5; border-radius: 12px; border: 1px solid #ff6b6b; color: #c62828; font-family: 'Segoe UI', Arial, sans-serif; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">
    <h2 style="margin-top: 0; color: #c62828;">Details Not Found</h2>
    <p style="font-size: 16px; margin: 15px 0;">
        The mobile number or the date of birth entered seems to be incorrect.
    </p>
    <p style="font-size: 14px; color: #8e0000;">
        Please double-check your information and try again.
    </p>
</div>

<%
	}
	%>
</div>
<%

}%>
```
---

# Send Mail using Zoho Creator Form with Zoho Sign Template :

```javascript

// Form input values
recipientName = input.Name;
recipientEmail = input.Email;

// API URL (your template)
url = "https://sign.zoho.com/api/v1/templates/30588000010568069/createdocument";

// Construct request body
dataMap = Map();

actionMap = Map();
actionMap.put("recipient_name", recipientName);
actionMap.put("recipient_email", recipientEmail);
actionMap.put("action_id", "30588000010568116");
actionMap.put("action_type", "SIGN");
actionMap.put("signing_order", 1);
actionMap.put("verify_recipient", false);
actionMap.put("private_notes", "");

actionsList = List();
actionsList.add(actionMap);

// Pass form data into document fields
fieldTextData = Map();
fieldTextData.put("Name", recipientName);
fieldTextData.put("Email", recipientEmail);

fieldData = Map();
fieldData.put("field_text_data", fieldTextData);
fieldData.put("field_boolean_data", Map());
fieldData.put("field_date_data", Map());
fieldData.put("field_radio_data", Map());
fieldData.put("field_checkboxgroup_data", Map());

// Template map
templateMap = Map();
templateMap.put("field_data", fieldData);
templateMap.put("notes", "Please sign this document");
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
    input.Document_ID = docId;
}

```
---

# Search in form and get report data :

```javascript
list = New_Reception[Reference_Code == "G176663"];
// walkin_list = New_Reception[Id1 == input.Id1] sort by Entry_Time1 desc range from 1 to 10;

// get one record data
for each rec in list
{
	info "Reference_Code: " + rec.Reference_Code;
	info "Mobile: " + rec.Mobile;
	info "id: " + rec.Id1;  
	info "Location: " + rec.Location;  
	info "Entry Time: " + rec.Entry_Time1;  
	break;
}

// For Table 
html = "<table border='1' style='border-collapse:collapse;width:1text-align:center font-family:sans-serif;'>";
html = html + "<tr style='background:#f2f2f2;'><th>Location</th><th>Entry Tith></tr>";

for each  rec in list
{
	location = ifnull(rec.Location,"-");
	entry_time = ifnull(rec.Entry_Time1,"-");
	html = html + "<tr>";
	html = html + "<td>" + location + "</td>";
	html = html + "<td>" + entry_time + "</td>";
	html = html + "</tr>";
}
html = html + "</table>";
input.plain = html;

```