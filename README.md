# Tableau Webhooks Documentation

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Set Up a Webhook Using Postman](#postman)
- [Set Up a Webhook Using cURL](#curl)
- [Events](#events)
- [Payloads](#payloads)
- [Tableau Server REST API Endpoints for Webhooks](#endpoints)
- [Tableau Webhooks Behavior](#behavior)

## <a id="introduction"></a>Introduction  

Webhooks let you build custom applications or workflows that react
to events that happen in Tableau. For example, you could use
webhooks to send an SMS or Slack notification any time
a datasource refresh fails, or fire off a confetti cannon when a new
workbook is created.  For the initial release,
webhooks are supported for a selected set of [datasource and workbook events](#events).  

You configure each webhook to subscribe to an event in Tableau. Then,
when the event occurs, an HTTP POST request will be sent to the public
URL you specified. This POST request includes a JSON payload that
includes information about the event. The payload includes the ID of the
object in question so that the Tableau REST API can be used to get
additional information or take further action. 

Learn more about [Tableau Webhooks Behavior](#behavior). 

## <a id="prerequisites"></a>Prerequisites  

To use Tableau webhooks, you must be authenticated as a site administrator for the Tableau Online or Server instance where the webhooks feature is enabled, for example, [https://10ax.online.tableau.com](https://10ax.online.tableau.com/). 

You can set up your own Tableau Online instance and get many other benefits at no cost by joining the [Tableau Developer Program](https://developer.tableau.com).  

## <a id="postman"></a>Set Up a Webhook Using Postman

Here is an example of setting up a webhook with the REST API
using Postman.

1. Download the following files (follow the link and save from your browser or IDE:

    - [Postman-Collection-Tableau-Webhooks.json](https://raw.githubusercontent.com/tableau/webhooks-docs/master/Postman-Collection-Tableau-Webhooks.json)

    - [Postman-Environment-Tableau-Webhooks.json](https://github.com/tableau/webhooks-docs/blob/master/Postman-Environment-Tableau-Webhooks.json)

1. Download and install Postman from [https://www.getpostman.com/](https://www.getpostman.com/).

1. Launch Postman and click **File** \> **Import** and then choose the Postman collection file you downloaded. 

1. Choose the **Tableau Webhooks Requests** collection on the left, and then select **Tableau Webhooks** from the environment dropdown menu on the top right.

1. To configure environment variables for webhooks, click the gear icon near the environment dropdown. In the **MANAGE ENVIRONMENTS** dialog, select **Tableau Webhooks**. Replace the placeholder URL for the `server` variable in the **CURRENT VARIABLE** column with your server URL (like `https://10ax.online.tableau.com`). Close the dialog.

1. In the **Tableau Webhooks Requests** collection, choose the **Sign-in** request. For Tableau Online or a named server site, add the `contentURL` of your site (like `my_site` in `https://10ax.online.tableau.com/site/my_site/projects`). Click **Send**. The response body contains the site id and a token.  

1. Open the **MANAGE ENVIRONMENTS** dialog from the gear icon, open **Tableau Webhooks** variables and use the site id and token to set **CURRENT VALUE** of the `site_id` and `tableau_auth_token` variables.

1. In the list of requests, click **Create a webhook**. In the request body, enter a webhook `name`, a  `webhook-source-api-event-name` from the [Events](#events) table, and a destination `url`. The destination URL must be https and have a valid certificate.

1. Click **Send**, and then use the ID of your new webhook from the response body to set the `webhook-id` environment variable in the **MANAGE ENVIRONMENTS** dialog.

1. In the list of requests, choose **Test a webhook** and click  **Send**. Testing the webhook sends an empty payload to the configured destination URL of the webhook and returns the response from the server. This is useful for testing, to ensure that webhooks POSTs are being sent from Tableau and a response is returnedfrom the destination as expected.

## <a id="curl"></a>Set Up a Webhook Using cURL

Here is an example of setting up a webhook with the REST API
using cURL.

### Sign In

`curl "http://<server>/api/3.1/auth/signin" -X POST -d @signin.xml`

Content of signin.xml:

```
<tsRequest>

  <credentials name="<username>" password="<password>" >

    <site contentUrl="" />

  </credentials>

</tsRequest>
```

### Create a Webhook

`curl "http://<server>/api/3.6/sites/<site-id>/webhooks" -X POST -H "X-Tableau-Auth:<token>" -d @details.xml`

Replace token with the token from the sign in response body.

Content of details.xml:

```
<tsRequest>

  <webhook name="my_first_webhook">

    <webhook-source>

      <webhook-source-event-datasource-created />

    </webhook-source>

    <webhook-destination>

      <webhook-destination-http method="POST" url="<URL>" />

    </webhook-destination>

  </webhook>

</tsRequest>
```

Replace URL with the destination URL for the webhook. The webhook destination URL must be https and have a valid certificate.

### Test the Webhook

Testing the webhook sends an empty payload to the configured destination
URL of the webhook and returns the response from the server. This is
useful for testing, to ensure that things are being sent from Tableau
and received back as expected.

`curl "http://<server>/api/3.6/sites/<site-id>/webhooks/<webhook-id>/test" -X GET -H "X-Tableau-Auth:<token>"`

Replace `webhook-id` with the webhook id from the create webhook response body.

See the [Test a Webhook](#testawebhook) endpoint for more information.

### List Webhooks

`curl "http://<server>/api/3.6/sites/<site-id>/webhooks" -X GET -H "X-Tableau-Auth:<token>"`

## <a id="events"></a>Events

For the initial release of webhooks, these events are supported:  

| Friendly Event Name          | API Event Name                                    |
| ---------------------------- | ------------------------------------------------- |
| DatasourceRefreshStarted   | webhook-source-event-datasource-refresh-started   |
| DatasourceRefreshSucceeded | webhook-source-event-datasource-refresh-succeeded |
| DatasourceRefreshFailed    | webhook-source-event-datasource-refresh-failed    |
| DatasourceUpdated           | webhook-source-event-datasource-updated           |
| DatasourceCreated           | webhook-source-event-datasource-created           |
| DatasourceDeleted           | webhook-source-event-datasource-deleted           |
| WorkbookUpdated             | webhook-source-event-workbook-updated             |
| WorkbookCreated             | webhook-source-event-workbook-created             |
| WorkbookDeleted             | webhook-source-event-workbook-deleted             |
| WorkbookRefreshStarted     | webhook-source-event-workbook-refresh-started     |
| WorkbookRefreshSucceeded   | webhook-source-event-workbook-refresh-succeeded |
| WorkbookRefreshFailed      | webhook-source-event-workbook-refresh-failed      | 

## <a id="payloads"></a>Payloads  

When one of the subscribed events fires, a JSON payload is sent to the
URL that is configured. The payloads vary based on the type of event.  

### Datasource Events  

The payloads for the datasource events (refresh started, refresh
succeeded, refresh failed, created, deleted, updated) are the same:  

```
{  

  "resource":"DATASOURCE",  

  "event_type":"DatasourceCreated",  

  "resource_name":"My Datasource",  

  "site_luid":"8b2a95d8-52b9-40a4-8712-cd6da771bd1b",  

  "resource_luid":"99",  

  "created_at":"2018-11-15T17:14:45Z"

}
```
| Field         | Description                                                                          |
| ------------------ | ----------------------------------------------------------------------------------------- |
| resource           | Will always be “DATASOURCE” for datasource events.                                        |
| event_type         | Type of event that occurred. Can be DatasourceRefreshStarted, DatasourceRefreshSucceeded, DatasourceRefreshFailed, DatasourceCreated, DatasourceDeleted, or DatasourceUpdated. |
| resource_name      | Name of the datasource in question.                                                       |
| site_luid            | LUID for the site that contains the datasource.                                           |
| resource_luid        | The datasource ID.                                                                        |

### Workbook Events  

The payloads for the workbook events (created, deleted, updated) are the
same:  

```
{  

  "resource":"WORKBOOK",  

  "event_type":"WorkbookCreated",  

  "resource_name":"My Workbook",  

  "site_luid":"8b2a95d8-52b9-40a4-8712-cd6da771bd1b",  

  "resource_luid":"99",  

  "created_at":"2018-11-15T17:14:45Z"

}

```

| Field         | Description                                                                          |
| ------------------ | ----------------------------------------------------------------------------------------- |
| resource           | Will always be “WORKBOOK” for workbook events.                                            |
| event_type         | Type of event that occurred. Can be WorkbookRefreshStarted, WorkbookRefreshSucceeded, and WorkbookRefreshFailed. |
| resource_name      | Name of the workbook in question.                                                         |
| site_luid            | LUID for the site that contains the workbook.                                             |
| resource_luid        | The workbook ID.                                                                          |

## <a id="endpoints"></a>Tableau Server REST API Endpoints for Webhooks  

### API Version  

All REST API endpoints for the initial release of webhooks are under the 3.6 API version. The base URL for this API version is: `https://{{server}}/api/3.6/`.

### Authentication  

The Tableau Server REST API requires that you send an authentication token, in the **X-Tableau-Auth** header, with each request. The token lets the server verify your identity and makes sure that you signed in. For more information, see [Signing In and Signing Out (Authentication)](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm).  

### Create a Webhook  

Creates a new webhook for a site.  

#### URI

`POST /api/3.6/sites/<site-id>/webhooks`

#### Parameter Values

`site-id` The ID of the site to create the webhook in.  
  
#### Request Body

```
<tsRequest>  
  <webhook name="webhook-name">  
    <webhook-source>  
      <webhook-source-api-event-name />  
    </webhook-source>  
    <webhook-destination>  
      <webhook-destination-http method="POST" url="url" />  
    </webhook-destination>
  </webhook>  
</tsRequest>
```

#### Attribute Values

`webhook-name`   A name for the webhook.

`webhook-source-api-event-name`   The API event name for the source event. It must be one of the supported events, such as, \<webhook-source-event-datasource-refresh-started />  

`url`   The destination URL for the webhook. The webhook destination URL must be https and have a valid certificate.
  
#### Response Code

`201`  

#### Response Body

```
<tsResponse>  
    <webhook id="webhook-id" name="webhook-name">  
        <webhook-source>  
            <webhook-source-api-event-name />  
        </webhook-source>  
        <webhook-destination>  
            <webhook-destination-http method="POST" url="url"/>  
        </webhook-destination>  
        <owner id="webhook_owner_luid" name="webhook_owner_name"/>
    </webhook>  
</tsResponse>
```

#### Response Headers

`Location: /api/<api-version>/sites/<site-id>/webhooks/<new-webhook-id>`  

### Get a Webhook  

Returns information about the specified webhook.  

#### URI

`GET /api/3.6/sites/<site-id>/webhooks/<webhook-id>`

#### Parameter Values

`site-id`   The ID of the site that contains the webhook.

`<webhook-id>`   The ID of the webhook to get information for.  
  
#### Request Body

None  

#### Response Code

`200`
  
#### Response Body

```
<tsResponse>  
    <webhook id="webhook-id" name="webhook-name">  
        <webhook-source>  
            <webhook-source-api-event-name />  
        </webhook-source>  
        <webhook-destination>  
            <webhook-destination-http method="POST" url="url"/>  
        </webhook-destination>  
        <owner id="webhook_owner_luid" name="webhook_owner_name"/>
    </webhook>  
</tsResponse>
```

### List Webhooks

Returns a list of all the webhooks on the specified site.  

#### URI

`GET /api/3.6/sites/<site-id>/webhooks`
  
#### Parameter Values

`site-id`   The ID of the site that contains the webhooks.  
  
#### Request Body

None  

#### Response Code

`200`

#### Response Body

```
<tsResponse>  
   <webhooks>  
      <webhook id="webhook-id" name="webhook-name">  
        <webhook-source>  
            <webhook-source-api-event-name />  
        </webhook-source>  
        <webhook-destination>  
            <webhook-destination-http method="POST" url="url"/>  
        </webhook-destination>
        <owner id="webhook_owner_luid" name="webhook_owner_name"/>
       </webhook>  
       <!--  ... additional webhooks ...  -->
   </webhooks>  
</tsResponse>  
```

### <a id="testawebhook"></a>Test a Webhook

Tests the specified webhook. Sends an empty payload to the configured destination URL of the webhook and returns the response from the server. This is useful for testing, to ensure that things are being sent from Tableau and received back as expected.  

#### URI

`GET /api/3.6/sites/<site-id>/webhooks/<webhook-id>/test`

#### Parameter Values

`site-id`   The ID of the site that contains the webhook.

`webhook-id`   The ID of the webhook to test.  

#### Request Body

None  

#### Response Code

`200`

A response in the `2xx` range means a successful test of the webhook. Responses in the `4xx` range mean that the webhook did not function properly, or does not exist (`400113`). 

#### Response Body

```
<tsResponse>  
    <webhookTestResult id="9f9bcaf8-8c4c-403c-b7e1-10dd85620f00" status="200">
       <body></body>  
    </webhookTestResult>  
</tsResponse>  
```

### Delete a Webhook  

Deletes the specified webhook.  

#### URI

`DELETE /api/3.6/sites/<site-id>/webhooks/<webhook-id>`

#### Parameter Values

`site-id`   The ID of the site that contains the webhook.  

`webhook-id`   The ID of the webhook to delete.  
  
#### Request Body

None  

#### Response Code

`204`

#### Response Body

None  

### Update a Webhook  

To modify a webhook after it has been created, delete it and recreate it.

## <a id="behavior"></a>Tableau Webhooks Behavior

- In some cases, a Tableau event may cause more than one webhook request to be sent to the destination URL server. We recommend that you parse incoming webhook requests to filter duplicates. The JSON payloads of duplicate requests will  be identical.
 
- When a server that has been sent a webhook request does not reply with a HTTP success code, the webhook will retry the request three times with diminishing frequency. (A HTTP success code is defined as any number in the [2xx range](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#2xx_Success).)

