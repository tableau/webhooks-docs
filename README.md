# Tableau Webhooks Documentation

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Set Up a Webhook Using Postman](#postman)
- [Set Up a Webhook Using cURL](#curl)
- [Trigger Events](#events)
- [Payloads](#payloads)
- [Tutorials](#tutorials)
  - [Set Up a Webhook with Glitch and If This The That](#tutorials)
  - [Set up a Webhook with Glitch and Slack](#tutorials)
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

1. Launch Postman and click **File** \> **Import** and choose the Postman collection and environment files you downloaded then select **Import**.

1. Choose the **Tableau Webhooks Requests** collection on the left, and then select **Tableau Webhooks** from the environment dropdown menu on the top right.

1. To configure environment variables for webhooks, click the sliders icon near the environment dropdown. In the **MANAGE ENVIRONMENTS** dialog, select **Tableau Webhooks**. Replace the placeholder URL for the `server` variable in the **CURRENT VARIABLE** column with your server URL (like `https://10ax.online.tableau.com`). Fill in the `content-url` environment variable for your site (content-url value would be `my_site` if your site url looks like this - `https://10ax.online.tableau.com/site/my_site/projects`). Add either your username and password or your Personal Access Token name and secret. Click **Update** and close the dialog.

1. In the **Tableau Webhooks Requests** collection, choose the **Sign-in** request. You can choose either the username/password method or the personal access token method.  Click **Send**. The response body contains the site id and a token.  

1. Open the **MANAGE ENVIRONMENTS** dialog from the sliders icon, open **Tableau Webhooks** variables, and use the site id and token to set **CURRENT VALUE** of the `site_id` and `tableau_auth_token` variables.

1. In the list of requests, click **Create a webhook**. In the request body, there are 3 environment variables you must fill out in order to send a valid request. In the environment variable management dialog, provide a value for `webhook-name`, `webhook-event` (see [Trigger Events Table](#events), and `webhook-url` (destination URL must be https and have a valid certificate).

1. Click **Send**, and then use the ID of your new webhook from the response body to set the `webhook-id` environment variable in the **MANAGE ENVIRONMENTS** dialog.

1. In the list of requests, choose **Test a webhook** and click  **Send**. Testing the webhook sends an empty payload to the configured destination URL of the webhook and returns the response from the server. This is useful for testing, to ensure that webhooks POSTs are being sent from Tableau and a response is returned from the destination as expected.

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

  <webhook name="my_first_webhook"   
    event="DatasourceRefreshStarted" >

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

## <a id="events"></a>Trigger Events

For the initial release of webhooks, the following events are supported. 

> Note that starting in Tableau 2020.3, the `event` attribute of your webhook is the preferred place to specify the triggering event. `webhook-source` can also be used or omitted, as long as there is no conflict between the event described in the two elements.    

| `event` Name          | `webhook-source` Name                                    |
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

## <a id="tutorials"></a>Webhooks Tutorials

### Set up a Tableau Webhook Using Glitch and If This Then That

In this tutorial, you can learn how to set up a [Glitch](http://glitch.com) project with [If This Then That](https://ifttt.com/) (IFTTT) as the service that processes your webhook message.

- [Send a message when a workbook is created](https://github.com/tableau/datadev-hackathon/wiki/Send-a-notification-when-a-workbook-is-created)

### Integrate Tableau Webhooks with Slack Using Glitch

In this tutorial, you can learn how to use a [Glitch](http://glitch.com) project to integrate your webhook message into [Slack](https://slack.com). Using these techniques can help you avoid commonly seen HTTP 400 errors.

- [Send a Slack notification when a workbook is created](https://github.com/tableau/datadev-hackathon/wiki/Send-a-Slack-notification-when-a-workbook-is-updated)

## <a id="endpoints"></a>Tableau Server REST API Endpoints for Webhooks  

### API Version  

Tableau REST API endpoints for managing your webhooks are available in API version 3.6 API. The base URL for this API version is: `https://{{server}}/api/{API_version_number}/`.

### Authentication  

The Tableau Server REST API requires that you send an authentication token, in the **X-Tableau-Auth** header, with each request. The token lets the server verify your identity and makes sure that you signed in. For more information, see [Signing In and Signing Out (Authentication)](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm).  

### **Create a Webhook**  

Creates a new webhook for a site.  

#### URI

```
POST /api/3.6/sites/<site-id>/webhooks
```

#### Parameter Values

`site-id` The ID of the site to create the webhook in.  
  
#### Request Body

```
<tsRequest>  
  <webhook 
    name="webhook-name" 
    isEnabled="true"  
    event="api-event-name">  
      <webhook-source>  
        <webhook-source-api-event-name />  
      </webhook-source>  
      <webhook-destination> 
        <webhook-destination-http method="POST" url="url"/>  
      </webhook-destination>
  </webhook>  
</tsRequest>
```

#### Attribute Values

`webhook-name`   Required. A name for the webhook.

`api-event-name`  | `webhook-source-api-event-name`   

_(One or the other is required.)_ The name of the Tableau event that triggers your webhook. The event name  must be one of the supported events listed in the [Trigger Events](#events) table. Note that `event` and `webhook-source` use different name values for the same event.  

> **Recommended**:
> Use the `event` attribute of the webhook to specify the triggering API event for the webhook. 
> 
> The `webhook-source` element serves the same purpose but is being deprecated in future versions of Tableau webhooks. 
> If both `events` and `webhook-source` are specified, their events specified must match. If either are specified, with the other being `NULL`, then the specified event becomes the webhook trigger, whether the element containing the event name is `event` or `webhook-source`. 

`url`   The destination URL for the webhook. The webhook destination URL must be https and have a valid certificate.

`webhook-enabled-flag`   Optional, boolean. If `true` (default), the newly created webhook is enabled. If `false` then the webhook will be disabled.
  
#### Response Code

`201` Success.

#### Response Body

```
<tsResponse>  
  <webhook 
    id="webhook-id" 
    name="webhook-name" 
    isEnabled="true"  
    statusChangeReason=""
    event="api-event-name">  
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

### **Get a Webhook**  

Returns information about the specified webhook.  

#### URI

```
GET /api/3.6/sites/<site-id>/webhooks/<webhook-id>
```

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
  <webhook 
    id="webhook-id" 
    name="webhook-name" 
    isEnabled="true"  
    statusChangeReason=""
    event="api-event-name">  
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

```
GET /api/3.6/sites/<site-id>/webhooks
```
  
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
      <webhook 
        id="webhook-id" 
        name="webhook-name" 
        isEnabled="true"  
        statusChangeReason=""
        event="api-event-name">  
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

### <a id="testawebhook"></a>**Test a Webhook**

Tests the specified webhook. Sends an empty payload to the configured destination URL of the webhook and returns the response from the server. This is useful for testing, to ensure that things are being sent from Tableau and received back as expected.  

#### URI

```
GET /api/3.6/sites/<site-id>/webhooks/<webhook-id>/test
```

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

### **Delete a Webhook**  

Deletes the specified webhook.  

#### URI

```
DELETE /api/3.6/sites/<site-id>/webhooks/<webhook-id>
```

#### Parameter Values

`site-id`   The ID of the site that contains the webhook.  

`webhook-id`   The ID of the webhook to delete.  
  
#### Request Body

None  

#### Response Code

`204`

#### Response Body

None  

### **Update a Webhook**  

Modify the properties of an existing webhook.

#### URI

```
PUT /api/3.8/sites/<site-id>/webhooks/<webhook-id>
```

#### Parameter Values

`site-id` The ID of the site that contains the webhook to be updated.  
  
`webhook-id` The ID of the webhook to be updated.  

#### Request Body

```
<tsRequest>  
  <webhook name="webhook-name" 
     isEnabled="webhook-enabled-flag" 
     statusChangeReason="reason-for-disablement"
     event="api-event-name">  
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

`webhook-name`   Required. A name for the webhook.


`api-event-name`  | `webhook-source-api-event-name`  

 Optional. The name of the Tableau event that triggers your webhook. You can update the trigger event for your webhook by specifying one of the supported events listed in the [Trigger Events](#events) table for either `event` or `webhook-source`. Note that `event` and `webhook-source` use different name values for the same event. 

> **Recommended**:
> Use the `event` attribute of the webhook to specify the triggering API event for the webhook. 
> 
> The `webhook-source` element serves the same purpose but is being deprecated in future versions of Tableau webhooks. 
> If both `events` and `webhook-source` are specified, the events specified must match. If either are specified, with the other being `NULL`, then the specified event becomes the webhook trigger, whether the element containing the event name is `event` or `webhook-source`. 


`url`   The destination URL for the webhook. The webhook destination URL must be https and have a valid certificate.

`webhook-enabled-flag`   Optional, boolean. If `true` (default), the updated webhook is enabled. If `false` then the webhook will be disabled.

`reason-for-disablement`   The reason a webhook is disabled. 

- If `isEnabled` set to `true`, omit this parameter from your request to create a webhook, or set the value of `statusChangeReason` to an empty string. Providing a reason value when creating an enabled webhook will result in an error (400127).
- If `isEnabled` set to `false`, then unless you provide a value for `statusChangeReason` it will default to "Webhook disabled by user".
  
#### Response Code

`201` Success.

`400` Error 400127: `statusChangeReason` was provided with `isEnabled` = `true` 

#### Response Body

```
<tsResponse>  
    <webhook 
      id="webhook-id" 
      name="webhook-name"  
      isEnabled="true" 
      statusChangeReason=""
      event="api-event-name">  
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

## <a id="behavior"></a>Tableau Webhooks Behavior

- In some cases, a Tableau event may cause more than one webhook request to be sent to the destination URL server. We recommend that you parse incoming webhook requests to filter duplicates. The JSON payloads of duplicate requests will  be identical.
 
- When a server that has been sent a webhook request does not reply with a HTTP success code, the webhook will retry the request three times with diminishing frequency. (A HTTP success code is defined as any number in the [2xx range](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#2xx_Success).)

