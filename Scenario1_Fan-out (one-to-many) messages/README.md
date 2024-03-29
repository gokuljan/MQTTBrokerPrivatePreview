# Scenario 1 – Fan-out (one-to-many) messages
This scenario simulates cloud-to-device commands to several devices and can be leveraged for use cases such as sending alerts to devices. Consider the use case where a fleet management service needs to send a weather alerts to all the vehicles in the fleet.

**Scenario:**

|Client | Role | Topic/Topic Filter|
| ------------ | ------------ | ------------ |
|Fleet-mgmt-device | Publisher | fleets/alerts/weather/alert1|
|Vehicle1 | Subscriber | fleets/alerts/#|
|Vehicle2 | Subscriber | fleets/alerts/#|

**Resource Configuration:**
|Client| Client Group| PermissionBinding (Role)| TopicSpaces|
| ------------ | ------------ | ------------ | ------------ |
|fleet-mgt-client (Attributes: “Type”:”Fleet-Mgmt”)| FleetMgmt| FleetMgmt-publisher|  WeatherAlerts (Topic template: fleet/alerts/weather/alert1) -Subscription Support: HighFanout|
|vehicle1 (Attributes: “Type”:”Vehicle”)| Vehicles| Vehicles-subscriber|  WeatherAlerts (Topic template: fleet/alerts/#) -Subscription Support: NotSupported|
|vehicle2 (Attributes: “Type”:”Vehicle”)| Vehicles| Vehicles-subscriber|  WeatherAlerts (Topic template: fleet/alerts/#) -Subscription Support: NotSupported|


![Deploy to Azure](https://aka.ms/deploytoazurebutton)


**High-level steps:**
- Register the following clients:
	- Fleet_mgmt_device
		- Attribute: Type=management
		- Authentication: self-signed certificate > PrimaryThumbprint
	- Vehicle1
		- Attribute: Type=vehicle
		- Authentication: self-signed certificate > PrimaryThumbprint
	- Vehicle2
		- Attribute: Type=vehicle
		- Authentication: self-signed certificate > PrimaryThumbprint
- Create the following client groups:
	- Fleet-mgmt to include the Fleet_mgmt_device client
		- Query: ${client.attribute.Type}= “management”
	- Vehicles to include vehicle1 and vehicle2 clients
		- Query: ${client.attribute.Type}= “vehicle”
- Create the following topic space:
	- WeatherAlerts
		- Topic Template: fleets/alerts/#
		- Subscription Support: HighFanout
- Create the following permission bindings:
	- FleetMgmt-Pub: to grant access for the client group Flee-mgmt to publish to the topic space WeatherAlerts
	- Vehicles-Sub: to grant access for the client group Vehicles to subscribe to the topic space WeatherAlerts



**CLI Instructions:**

Download the folder Scenario1_jsons with JSON files to C:

Replace the \<Subscription ID\> with your subscription ID in below commands.

Create the namespace under the resource group that you already created as part of the prerequisites.

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1 --is-full-object --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\NS_Scenario1.json
```

If you are using a CA certificate to authenticate the clients, the encoded certificate string must be in valid PEM (Privacy Enhanced Mail) format with header (-----BEGIN CERTIFICATE-----) and footer (-----END CERTIFICATE-----). This string must not include a private key. Save the certificate as a json file named MqttCACertificate.json in C:\Scenario1_jsons\ folder.  You can include a description in the properties as below.

```json
{
    "properties":{
   	    "description": "This is a CA certificate",
        "encodedCertificate": "-----BEGIN CERTIFICATE-----
			---Base64 encoded Certificate---
 -----END CERTIFICATE-----"
    }
}
```

Register the CA Certificate using the below command.  

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces/caCertificates --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1/caCertificates/CACert --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\MqttCACertificate.json
```

Onboard the Clients using below CLI commands.

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces/clients --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1/clients/fleet_mgt_client --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\C_fleet_mgt_client.json
```

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces/clients --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1/clients/vehicle1 --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\C_vehicle1.json
```

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces/clients --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1/clients/vehicle2 --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\C_vehicle2.json
```

Create the Client Groups using below CLI commands

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces/clientGroups --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1/clientGroups/FleetMgmt --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\CG_FleetMgmt.json
```

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces/clientGroups --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1/clientGroups/Vehicles --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\CG_Vehicles.json
```

Create the Topic Spaces using below CLI commands

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces/topicSpaces --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1/topicSpaces/WeatherAlerts --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\TS_WeatherAlerts.json
```

Create the Permission Bindings using below CLI commands

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces/permissionBindings --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1/permissionBindings/FleetMgmt-publisher --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\PB_FleetMgmt-publisher.json
```

```bash
az resource create --resource-type Microsoft.EventGrid/namespaces/permissionBindings --id /subscriptions/<Subscription ID>/resourceGroups/MQTT-Pri-Prev-rg1/providers/Microsoft.EventGrid/namespaces/Scenario1/permissionBindings/Vehicles-subscriber --api-version 2022-10-15-preview --properties @C:\Scenario1_jsons\PB_Vehicles-subscriber.json
```


**Instructions to deploy using ARM template on portal:**

1. Go to Azure portal, type "deploy a custom template" in the search.

<img src="Deploy ARM template on portal 1.png"
     alt="Deploy ARM template on portal 1"
     style="float: left; margin-right: 10px;" />


2. Select "Deploy a custom template" from the Services list.  Click on "Build your own template in the editor".

<img src="Deploy ARM template on portal 2.png"
     alt="Deploy ARM template on portal 2"
     style="float: left; margin-right: 10px;" />

3. Copy the json from the "ARM template for all resources.json" file in the scenario's json folder into the editor.  Click the save button.
	 
<img src="Deploy ARM template on portal 3.png"
     alt="Deploy ARM template on portal 3"
     style="float: left; margin-right: 10px;" />

4. Check the subscription and select the resource group.  Click on the "Reveiw + Create" button to initiate the deployment.

<img src="Deploy ARM template on portal 4.png"
     alt="Deploy ARM template on portal 4"
     style="float: left; margin-right: 10px;" />

5. You will see the screen that deployment is in progress, wait till it's successful.  Once it shows the deployment is successful, you can see the list of all the resources created for the scenario.

<img src="Deploy ARM template on portal 5.png"
     alt="Deploy ARM template on portal 5"
     style="float: left; margin-right: 10px;" />

