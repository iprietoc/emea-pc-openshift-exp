# OpenShift Experience Session Materials for Red Hat Partner Conference EMEA 2019

## Lab Details

* Cluster: http://console-openshift-console.apps.cluster-2505.2505.events.opentlc.com/
* User Selection: https://etherpad.net/p/Openshift_experience

## Lab

### Connecting
The cluster was provisioned using a self-signed certificate. Ignore the privacy errors and proceed to the console.

### Create a Project
* Click the "Create Project" button
* Under the Name field, enter `pcdemo` 

### Deploy MySQL
* Use the Catalog -> Developer Catalog menu to find the “MySQL (Ephemeral)" entry
* Click the icon and press the “Create Service Instance” button
* On the configuration screen, leave all of the values as the default except for:
  * MySQL Connection Username: `rayner`
  * MySQL Connection Password: `rayner`
  * MySQL Database Name: `rayner`
* Click the “Create” button.

### Deploy the Rayner Service
* Using the “Add” button in the top right of the UI, select the “Deploy Image” menu item
* Under “Image Name”, enter `jdob/rayner-service:latest` and press Enter
  * Leave the “Name” to the default of “rayner-service”
  * Under Environment Variables, enter the following:
    * MYSQL_DATABASE: `rayner`   (this must match MySQL Database Name from above)
    * BRIDGE_MOCK: `true`

_The presence of `MYSQL_DATABASE` tells the Rayner Service to use MySQL instead of the Django default of sqlite. The rest of the connection details, such as username and host, are still needed to find and connect to the database. For simplicity, the rayner-service defaults those value._

_Setting BRIDGE_MOCK to true makes it so that the service won’t try to contact the bulb and will just log the calls to the database. If this is omitted, any color change request into the REST API will take a while and eventually time out._

### Create a Route to Rayner Service
* Select the Networking -> Routes menu item 
* Press the "Create Route" button
* Under "Name" enter "rayner-service"
* Under "Service" select "rayner-service"
* Press the "Create" button

### Deploy Rayner UI
* Similar to the previous step, the web UI will be deployed through the “Add” menu and selecting “Deploy Image”. 
* Under “Image Name”, enter `jdob/rayner-webui:latest` and press Enter
  * Leave the “Name” to the default of “rayner-webui”
  * Under Environment Variables, enter the following:
    * REACT_APP_SERVICE_HOST `$HOST` (see note below)
    * REACT_APP_SERVICE_PORT=80

*Important: Make sure the value of $HOST is only the hostname of the rayner-service. Do not include the “http:/” or the trailing “/”.*

### Create a Route to Rayner Web UI
* Select the Networking -> Routes menu item 
* Press the "Create Route" button
* Under "Name" enter "rayner-webui"
* Under "Service" select "rayner-webui"
* Press the "Create" button

On the resulting screen, click the URL under the Location column for the entry named "rayner-webui".

### Send a Sample Request
* In order to get sample data into the web UI, requests must be sent to the backend service. This can easily be accomplished using “curl” from the terminal. In the example below, $SERVICE_ROUTE is the full URL (including http://) to the *rayner-service* route.

`curl -X PUT -H "Content-Type: application/json" -d '{"hex": "00ff00"}' $SERVICE_ROUTE/light/`

_Note that the trailing `/` on the URL is required._

### Scale the Rayner Service
* Select the “Home” -> “Projects” entry from the menu on the left and select the "pcdemo" project from the resulting screen. Three services should be listed in the “Project Status” screen, one for each of the previous steps.
* Click the name of the “rayner-service” entry. A flyout menu will appear on the right.
* Under “Desired Count”, click the pencil icon.
* Use the plus button (or simply type in the value) to set the number of pods to 2 and press Save.

The status of the scaling operation will appear in the flyout menu. In particular, pay attention to the list of available and unavailable pods. When the count reads “2 available”, the scale is finished.

### Run the Load Testing Job
* From the left menu, navigate to “Workloads” > “Jobs”.
* Press the “Create Job” button.
* Replace the provided YAML with the contents of: 
https://raw.githubusercontent.com/jdob/rayner-bot/master/openshift/job.yaml
* Press the “Create” button.

There are two places to watch. From the screen that resulted from creating the Job, click the “Pods” tab. That will show the details of the pods being created as part of the job. Notice how there are never more than 3 in the “Running” state at a time.

Switch to the rayner web UI tab. Requests should appear as the jobs issue them against the service API. Notice how the “Service IP” field alternates between two IPs, indicating the requests are being load balanced across the two service containers.
