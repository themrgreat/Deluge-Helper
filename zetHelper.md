### Related Repository
[📄 Transcript Viewer](https://github.com/MrGreat-0/Transcript_Viewer)  
A custom Zoho CRM widget for parsing and visualizing JSON-based call transcripts.

[📄 Manage Leave Request](https://github.com/MrGreat-0/Manage_Leave_Request)  
Zoho CRM widget for managing and bulk-approving leave requests from the Portal_People module.

[📄 Partner Search](https://github.com/MrGreat-0/Partner-Search)
A custom Zoho CRM widget for discovering and filtering potential partners with structured search and management capabilities.

[📄 Weekly Plan](https://github.com/themrgreat/Weekly-Plan)
Zoho CRM widget for weekly School/Dealer visit planning with holiday-aware scheduling and automated Weekly Planner/Meeting record creation.


### Documentation

- [Zoho CRM Widgets Overview](https://www.zoho.com/crm/developer/docs/widgets/)

- [Installing the Zoho CLI](https://www.zoho.com/crm/developer/docs/widgets/install-cli.html)

- [Zoho Widgets JS SDK Documentation](https://help.zwidgets.com/help/latest/index.html)

- [Zoho CRM JS SDK Initialization Guide](https://www.zohocrm.dev/explore/widgets/latest/jssdk#init)

- [Create a Zoho CRM Widget](https://www.zoho.com/crm/developer/docs/widgets/create-widget.html)


---

# Base HTML Structure for Zoho CRM Widget :

```html
<!DOCTYPE html>
<html>

<head>
    <link rel="stylesheet" href="./widget.css">
</head>

<body>

    <div class="container">
        <!-- here you can write... -->
    </div>

    <script src="https://live.zwidgets.com/js-sdk/1.2/ZohoEmbededAppSDK.min.js"></script>
    <script src="./widget.js"></script>

</body>

</html>
```

---

# Zoho CRM Widget JavaScript Setup — Async/Await :

```javascript

console.log("......First......");

ZOHO.embeddedApp.on("PageLoad", async function (data) {
    try {
        console.log("PageLoad data:", data);

        const taskId = data.EntityId;
        const module = data.Entity;

        if (!taskId || !module) {
            throw new Error("Missing CRM context");
        }

        // await ONLY the API call
        const response = await ZOHO.CRM.API.getRecord({
            Entity: module,
            RecordID: taskId
        });

        console.log("CRM Response:", response);

        const description = response.data[0].Description;
        // renderChat(description);

    } catch (error) {
        console.error("Widget error:", error);
        document.getElementById("title").innerText =
            "Failed to load conversation";
    }
});

ZOHO.embeddedApp.init();

console.log("......Last......");

```

---

# Zoho CRM Widget JavaScript Setup (Promise `.then()` Version) :

```javascript

console.log("......First......");

ZOHO.embeddedApp.on("PageLoad", function (data) {

    const taskId = data.EntityId;
    const module = data.Entity;

    ZOHO.CRM.API.getRecord({
        Entity: module,
        RecordID: taskId
    })
    .then(function (response) {
        console.log(response);
    })
    .catch(function (error) {
        console.error(error);
    });

});

ZOHO.embeddedApp.init();

console.log("......Last......");

```

---