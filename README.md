## Getting Started with the Microsoft Power Query Connection to apaleo API

If you need to extract apaleo's data through its public API by using the Microsoft Power Query Connection, then this quick how-to will guide you through the steps. The apaleo APIs use [OAuth 2.0](https://www.oauth.com/oauth2-servers/map-oauth-2-0-specs/) to authenticate and authorize users to make requests. 



## What data are we getting from the API?

You might be mainly interested in the property performance API, which is documented here https://api.apaleo.com/swagger/index.html?urls.primaryName=Reports%20V1.

It provides you key performance indicator for the selected property like occupancy, ADR, and RevPAR.

You can also get data from other API endpoints based on your needs.



## Create a client app and get client credentials

The first step of creating an app is to register your new app at [New custom app](https://app.apaleo.com/apps/connected-apps/create) in apaleo.

The registration process involves entering basic client app details and the scopes that you require for the endpoints that you want to access.

If desired, you can edit a registered client app at any point in the future. The registration process is not considered as a part of the authorization flow. 

To learn how to create your client, go to [Register the OAuth simple client application](https://apaleo.dev/guides/start/oauth-connection/register-app#register-the-oauth-simple-client-application). Review your app details and save your app. Once you save the app, you'll get the following dialog-box.

<img src="/images/credentials.png"/>

> Take note of the client ID and client secret. You’ll need these in the [next step](https://github.com/apaleo/powerquery-reports#setting-up-the-api-connection-in-power-query) to initiate the OAuth flow.



## Setting up the API connection in Power Query

1. Start with a Blank Query. In this example, we are using Microsoft Excel.

<img src="/images/blank_query.png"/>

2. Open the Advanced Editor.

<img src="/images/advanced_editor.png"/>



The first part of the query requests an access token from the API. Insert your Client ID and Secret as shown below.

- [replace](https://github.com/apaleo/powerquery-reports/blob/master/query#L7) `APALEO_CLIENT_ID` and `APALEO_CLIENT_SECRET`

The second part of the query uses the access token that we've just generated to get the data from API.



```
  // Uses the api

TodayDate = DateTime.Date(DateTime.LocalNow()),

StartDate = (x as number) => Date.ToText(Date.StartOfWeek(Date.AddDays(TodayDate,x*7), Day.Monday),"yyyy-MM-dd"),
EndDate = (x as number) => Date.ToText(Date.EndOfWeek(Date.AddDays(TodayDate,x*7), Day.Monday),"yyyy-MM-dd"),

 GetJsonQuery = (x as number) => Web.Contents("https://api.apaleo.com/reports/v1/reports/ordered-services?propertyId=BER&serviceIds=MUC-BRK&from="&StartDate(x)&"&to="&EndDate(x),

     [

         Headers = [#"Authorization"=AccessTokenHeader]

     ]

 ),
```



The remainder of the query consists of various transformation steps to render your data. You can create your own transformations to suit your needs.



```
orderedServicesResponseTable = Record.ToTable(Json.Document(GetJsonQuery(1))),
orderedServicesList = orderedServicesResponseTable{0}[Value],
orderedServicesTable = Table.FromList(orderedServicesList, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
orderedServicesExpandedTable = Table.ExpandRecordColumn(orderedServicesTable, "Column1", {"id", "code", "name", "serviceDate", "count"}, {"ID", "Code", "Name", "ServiceDate", "Count"})


in
    orderedServicesExpandedTable
```

