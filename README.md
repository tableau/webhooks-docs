# Webhooks Documentation (Developer Preview)

## Introduction  

Webhooks let you build custom applications or workflows that react
to events that happen in Tableau Online. For example, you could use
webhooks to send an SMS or Slack notification any time
a datasource refresh fails, or fire off a confetti cannon when a new
workbook is created.  For the initial release of this developer preview,
webhooks are supported for datasource and workbook events only.  

You configure each webhook to subscribe to an event in Tableau. Then,
when the event occurs, an HTTP POST request will be sent to the public
URL you specified. This POST request includes a JSON payload that
includes information about the event. The payload includes the ID of the
object in question so that the Tableau REST API can be used to get
additional information or take further action.  

## Prerequisites  

To use Tableau webhooks, you must:

- enroll in the developer preview of the webhooks feature.  

- connect to the Tableau Online instance where the webhooks feature is enabled, [https://10ax.online.tableau.com](https://10ax.online.tableau.com/).  

- be a site administrator.

## Set Up a Webhook Using Postman

Here is an example of setting up a webhook with the REST API
using Postman.

1. Download the file [Postman Collection - Tableau Webhooks.json](tbd)

1. Download Postman from [https://www.getpostman.com/](https://www.getpostman.com/)

1. Launch Postman.

1. Click **File** \> **Import** and choose the Postman collection you downloaded. The collection appears on the left.

1. To configure the variables in the collection, click the ellipsis beside the collection name in the left sidebar.

1. Click **Edit**.

1. Click the **Variables** tab, change the Tableau\_Server variable to the correct server. Click **Update**.

1. In the list of requests, click **Sign-in** and then click **Send**. The response body contains the site id and a token.  

1. Click the **Variables** tab and set the Site\_ID and Tableau\_Auth\_Token variables.

1. In the list of requests, click **Create a webhook**. In the request body, change the webhook name and url. Click **Send**. The ID of the new webhook is returned in the response body.

1. In the list of requests, click **Test a webhook**. Set the webhook ID to test in the variables or in the URI and then click **Send**. Testing the webhook sends an empty payload to the configured destination URL of the webhook and returns the response from the server. This is useful for testing, to ensure that things are being sent from Tableau and received back as expected.

## Set Up a Webhook Using cURL

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

`curl "http://<server>/api/exp/sites/<site-id>/webhooks" -X POST -H "X-Tableau-Auth:<token>" -d @details.xml`

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

Replace URL with the destination URL for the webhook.

### Test the Webhook

Testing the webhook sends an empty payload to the configured destination
URL of the webhook and returns the response from the server. This is
useful for testing, to ensure that things are being sent from Tableau
and received back as expected.

`curl "http://<server>/api/exp/sites/<site-id>/webhooks/<webhook-id>" -X GET -H "X-Tableau-Auth:<token>"`

Replace webhook-id with the webhook id from the create webhook response body.

### List Webhooks

`curl "http://<server>/api/exp/sites/<site-id>/webhooks" -X GET -H "X-Tableau-Auth:<token>"`

## Events

For the initial release of the developer preview of webhooks, these events are supported:  

| Friendly Event Name          | API Event Name                                    |
| ---------------------------- | ------------------------------------------------- |
| Datasource Refresh Started   | webhook-source-event-datasource-refresh-started   |
| Datasource Refresh Succeeded | webhook-source-event-datasource-refresh-succeeded |
| Datasource Refresh Failed    | webhook-source-event-datasource-refresh-failed    |
| Datasource Updated           | webhook-source-event-datasource-updated           |
| Datasource Created           | webhook-source-event-datasource-created           |
| Datasource Deleted           | webhook-source-event-datasource-deleted           |
| Workbook Updated             | webhook-source-event-workbook-updated             |
| Workbook Created             | webhook-source-event-workbook-created             |
| Workbook Deleted             | webhook-source-event-workbook-deleted             |

## Payloads  

When one of the subscribed events fires, a JSON payload is sent to the
URL that is configured. The payloads vary based on the type of event.  

### Datasource Events  

The payloads for the datasource events (refresh started, refresh
succeeded, refresh failed, created, deleted, updated) are the same:  

```
{  

  "resource":"DATASOURCE",  

  "event-type":"DatasourceCreated",  

  "resource-name":"My Datasource",  

  "site-id":"8b2a95d8-52b9-40a4-8712-cd6da771bd1b",  

  "resource-id":"99"  

}
```

<table>
<thead>
<tr class="header">
<th><strong>Field </strong> </th>
<th><strong>Description </strong> </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>resource </strong> </td>
<td>Will always be “DATASOURCE” for datasource events.  </td>
</tr>
<tr class="even">
<td><strong>event-type </strong> </td>
<td><p>Type of event that occurred. Can be one of:  </p>
<ul>
<li>
<p>DatasourceRefreshStarted</p>
</li>
<li>
<p>DatasourceRefreshSucceeded</p>
</li>
<li>
<p>DatasourceRefreshFailed</p>
</li>
<li>
<p>DatasourceCreated</p>
</li>
<li>
<p>DatasourceDeleted</p>
</li>
<li>
<p>DatasourceUpdated</p>
</li>
</ul></td>
</tr>
<tr class="odd">
<td><strong>resource-name </strong> </td>
<td>Name of the datasource in question.  </td>
</tr>
<tr class="even">
<td><strong>site-id </strong> </td>
<td>LUID for the site that contains the datasource.  </td>
</tr>
<tr class="odd">
<td><strong>resource-id </strong> </td>
<td>The datasource ID.  </td>
</tr>
</tbody>
</table>

### Workbook Events  

The payloads for the workbook events (created, deleted, updated) are the
same:  

```
{  

  "resource":"WORKBOOK",  

  "event-type":"WorkbookCreated",  

  "resource-name":"My Workbook",  

  "site-id":"8b2a95d8-52b9-40a4-8712-cd6da771bd1b",  

  "resource-id":"99"  

}

```

| Field         | Description                                                                          |
| ------------------ | ----------------------------------------------------------------------------------------- |
| resource           | Will always be “WORKBOOK” for workbook events.                                            |
| event-type         | Type of event that occurred. Can be WorkbookCreated, WorkbookDeleted, or WorkbookUpdated. |
| resource-name      | Name of the workbook in question.                                                         |
| site-id            | LUID for the site that contains the workbook.                                             |
| resource-id        | The workbook ID.                                                                          |

## Tableau REST API Endpoints for Webhooks  

### API Version  

All REST API endpoints for webhooks are under the new experimental API version, “exp”. The base URL for the experimental API is: `https://10ax.online.tableau.com/api/exp/`.

### Authentication  

The Tableau Server REST API requires that you send an authentication token, in the **X-Tableau-Auth** header, with each request. The token lets the server verify your identity and makes sure that you signed in. For more information, see [Signing In and Signing Out (Authentication)](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm).  

### Create a Webhook  

Creates a new webhook for a site.  

<table>
<thead>
<tr class="header">
<th>  <br />
<strong>URI</strong>  </th>
<th>POST /api/<em>exp</em>/sites/<em>site-id</em>/webhooks  </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Parameter Values</strong>  </td>
<td><table>
<tbody>
<tr class="odd">
<td><em>site-id</em>  </td>
<td>The ID of the site to create the webhook in.  </td>
</tr>
</tbody>
</table>
<p>  </p></td>
</tr>
<tr class="even">
<td><strong>Request Body</strong>  </td>
<td><p>&lt;tsRequest&gt;  </p>
<p>  &lt;webhook name=&quot;<em><strong>webhook-name</strong></em>&quot;&gt;  </p>
<p>    &lt;webhook-source&gt;  </p>
<p>      &lt;<em><strong>webhook-source-event-name</strong></em> /&gt;  </p>
<p>    &lt;/webhook-source&gt;  </p>
<p>    &lt;webhook-destination&gt;  </p>
<p>      &lt;webhook-destination-http method=&quot;POST&quot; url=&quot;<em><strong>url</strong></em>&quot; /&gt;  </p>
<p>    &lt;/webhook-destination&gt;  </p>
<p>  &lt;/webhook&gt;  </p>
<p>&lt;/tsRequest&gt;  </p></td>
</tr>
<tr class="odd">
<td><strong>Attribute Values</strong>  </td>
<td><table>
<thead>
<tr class="header">
<th><em>webhook-name</em>  </th>
<th>A name for the webhook.   </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><em>webhook-source-event-name</em>  </td>
<td>The API event name for the source event. It must be one of the supported events, such as, &lt;webhook-source-event-datasource-refresh-started /&gt;  </td>
</tr>
<tr class="even">
<td><em>url</em>  </td>
<td>The destination URL for the webhook.  </td>
</tr>
</tbody>
</table>
<p>  </p></td>
</tr>
<tr class="even">
<td><strong>Response Code</strong>  </td>
<td>201  </td>
</tr>
<tr class="odd">
<td><strong>Response Body</strong>  </td>
<td><p>&lt;tsResponse&gt;  </p>
<p>    &lt;webhook id=&quot;<em>webhook-id</em>&quot; name=&quot;<em>webhook-name</em>&quot;&gt;  </p>
<p>        &lt;webhook-source&gt;  </p>
<p>            &lt;<em>webhook-source-event-name</em> /&gt;  </p>
<p>        &lt;/webhook-source&gt;  </p>
<p>        &lt;webhook-destination&gt;  </p>
<p>            &lt;webhook-destination-http method=&quot;POST&quot; url=&quot;<em><strong>url</strong></em>&quot;/&gt;  </p>
<p>        &lt;/webhook-destination&gt;  </p>
<p>    &lt;/webhook&gt;  </p>
<p>&lt;/tsResponse&gt;  </p></td>
</tr>
<tr class="even">
<td><strong>Response Headers</strong>  </td>
<td>Location: /api/<em>api-version</em>/sites/<em>site-id</em>/webhooks/<em>new-webhook-id</em>  </td>
</tr>
<tr class="odd">
<td>  </td>
<td><p>  </p>
<p>  </p></td>
</tr>
</tbody>
</table>

### Get a Webhook  

Returns information about the specified webhook.  

<table>
<thead>
<tr class="header">
<th>  <br />
<strong>URI</strong>  </th>
<th>GET /api/<em>exp</em>/sites/<em>site-id</em>/webhooks/<em>webhook-id</em>  </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Parameter Values</strong>  </td>
<td><table>
<thead>
<tr class="header">
<th><em>site-id</em>  </th>
<th>The ID of the site that contains the webhook.  </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><em>webhook-id</em>  </td>
<td>The ID of the webhook to get information for.  </td>
</tr>
</tbody>
</table>
<p>  </p></td>
</tr>
<tr class="even">
<td><strong>Request Body</strong>  </td>
<td>None  </td>
</tr>
<tr class="odd">
<td><strong>Response Code</strong>  </td>
<td>200  </td>
</tr>
<tr class="even">
<td><strong>Response Body</strong>  </td>
<td><p>&lt;tsResponse&gt;  </p>
<p>    &lt;webhook id=&quot;<em>webhook-id</em>&quot; name=&quot;<em>webhook-name</em>&quot;&gt;  </p>
<p>        &lt;webhook-source&gt;  </p>
<p>            &lt;<em>webhook-source-event-name</em> /&gt;  </p>
<p>        &lt;/webhook-source&gt;  </p>
<p>        &lt;webhook-destination&gt;  </p>
<p>            &lt;webhook-destination-http method=&quot;POST&quot; url=&quot;<em>url</em>&quot;/&gt;  </p>
<p>        &lt;/webhook-destination&gt;  </p>
<p>    &lt;/webhook&gt;  </p>
<p>&lt;/tsResponse&gt;  </p></td>
</tr>
<tr class="odd">
<td>  </td>
<td>  </td>
</tr>
</tbody>
</table>

### List Webhooks

Returns a list of all the webhooks on the specified site.  

<table>
<thead>
<tr class="header">
<th>  <br />
<strong>URI</strong>  </th>
<th>GET /api/<em>exp</em>/sites/<em>site-id</em>/webhooks  </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Parameter Values</strong>  </td>
<td><table>
<tbody>
<tr class="odd">
<td><em>site-id</em>  </td>
<td>The ID of the site that contains the webhooks.  </td>
</tr>
</tbody>
</table>
<p>  </p></td>
</tr>
<tr class="even">
<td><strong>Request Body</strong>  </td>
<td>None  </td>
</tr>
<tr class="odd">
<td><strong>Response Code</strong>  </td>
<td>200  </td>
</tr>
<tr class="even">
<td><strong>Response Body</strong>  </td>
<td><p>&lt;tsResponse&gt;  </p>
<p>   &lt;webhooks&gt;  </p>
<p>      &lt;webhook id=&quot;<em>webhook-id</em>&quot; name=&quot;<em>webhook-name</em>&quot;&gt;  </p>
<p>        &lt;webhook-source&gt;  </p>
<p>            &lt;<em>webhook-source-event-name</em> /&gt;  </p>
<p>        &lt;/webhook-source&gt;  </p>
<p>        &lt;webhook-destination&gt;  </p>
<p>            &lt;webhook-destination-http method=&quot;POST&quot; url=&quot;<em>url</em>&quot;/&gt;  </p>
<p>        &lt;/webhook-destination&gt;  </p>
<p>       &lt;/webhook&gt;  </p>
<p>      ... additional webhooks ...  </p>
<p>   &lt;/webhooks&gt;  </p>
<p>&lt;/tsResponse&gt;  </p></td>
</tr>
<tr class="odd">
<td>  </td>
<td>  </td>
</tr>
</tbody>
</table>

### Test a Webhook  

Tests the specified webhook. Sends an empty payload to the configured
destination URL of the webhook and returns the response
from the server. This is useful for testing, to ensure that things
are being sent from Tableau and received back as expected.  

<table>
<thead>
<tr class="header">
<th>  <br />
<strong>URI</strong>  </th>
<th>GET /api/<em>exp</em>/sites/<em>site-id</em>/webhooks/<em>webhook-id/</em>test  </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Parameter Values</strong>  </td>
<td><table>
<thead>
<tr class="header">
<th><em>site-id</em>  </th>
<th>The ID of the site that contains the webhook.  </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><em>webhook-id</em>  </td>
<td>The ID of the webhook to test.  </td>
</tr>
</tbody>
</table>
<p>  </p></td>
</tr>
<tr class="even">
<td><strong>Request Body</strong>  </td>
<td>None  </td>
</tr>
<tr class="odd">
<td><strong>Response Code</strong>  </td>
<td>200  </td>
</tr>
<tr class="even">
<td><strong>Response Body</strong>  </td>
<td><p>&lt;tsResponse&gt;  </p>
<p>    &lt;webhookTestResult id=&quot;9f9bcaf8-8c4c-403c-b7e1-10dd85620f00&quot; status=&quot;200&quot;&gt;      </p>
<p>       &lt;body&gt;&lt;/body&gt;  </p>
<p>    &lt;/webhookTestResult&gt;  </p>
<p>&lt;/tsResponse&gt;  </p></td>
</tr>
</tbody>
</table>

### Delete a Webhook  

Deletes the specified webhook.  

<table>
<thead>
<tr class="header">
<th>  <br />
<strong>URI</strong>  </th>
<th>DELETE /api/<em>exp</em>/sites/<em>site-id</em>/webhooks/<em>webhook-id</em>  </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Parameter Values</strong>  </td>
<td><table>
<thead>
<tr class="header">
<th><em>site-id</em>  </th>
<th>The ID of the site that contains the webhook.  </th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><em>webhook-id</em>  </td>
<td>The ID of the webhook to delete.  </td>
</tr>
</tbody>
</table>
<p>  </p></td>
</tr>
<tr class="even">
<td><strong>Request Body</strong>  </td>
<td>None  </td>
</tr>
<tr class="odd">
<td><strong>Response Code</strong>  </td>
<td>204  </td>
</tr>
<tr class="even">
<td><strong>Response Body</strong>  </td>
<td>None  </td>
</tr>
<tr class="odd">
<td>  </td>
<td>  </td>
</tr>
</tbody>
</table>

### Update a Webhook  

To modify a webhook after it has been created, delete it and recreate it.
