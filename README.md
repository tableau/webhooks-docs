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
