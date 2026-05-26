# Mass status update to Closed :
```javascript
orgId = "60031797391";

ticket_ids = {"AG1436420526","AG1437020526","AG1412920426","AG1437200526","AG1414630426","AG1438250526","AG1377220226","AG1377780226"};

for each single_id in ticket_ids
{
	searchParams = Map();
	searchParams.put("ticketNumber", single_id);

	searchResults = zoho.desk.searchRecords(orgId, "tickets", searchParams, 0, 1, "desk");

	if(searchResults.get("data").size() > 0)
	{
		ticketId = searchResults.get("data").get(0).get("id");

		recordValue = Map();
		recordValue.put("status", "Closed");

		response = zoho.desk.update(orgId, "tickets", ticketId, recordValue, "desk");
		info response;
	}
}
```

---