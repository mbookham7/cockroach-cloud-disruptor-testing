# Cockroach Cloud Disruptor Testing

In this repo the process for testing the resilience of Cockroach Cloud is documented. This involves isolating a single node or region and seeing the impact on the cluster. This process is driven by a feature flag and via the API and is not available via the UI. To test this feature you will need to have the feature flag enabled, talk to your Cockroach Labs account team to have this switched on. 

As there is no access via the UI an API key will need to be created.

1. In the Cloud Console click on `Organization` then `Access Management` for the dropdown menu. Click `Service Accounts` and then `Create`. Give the account a name and a short description and click create. Then give the API Key itself a name and hit create. Copy the API Secret Key and keep this safe as it is needed in further steps.

1. The Service Account will need Cluster Admin to the cluster that the testing is taking place on. In the `Service Accounts` list find the Service Account you just created, click the `Action` ellipsis and click `Edit Roles`. Click `New role`. In scope add the cluster that is being testing, under role select `Cluster Admin`. Now the Service Account has the correct permissions to perform the test.

1. To make further steps easier add the API Secret Key to an environment variable. This is the `Secret Key` we copied and kept safe from the earlier step.

```
export TOKEN=<add-api-secret-key-here>
```

Test the varable by echoing this out in the terminal.

```
echo $TOKEN
```

1. In this repo there is a file called `disruption_single_node.json` this is an example file we need to update this to reflect the region and pod names for the cluster we are testing.

Example file.
```
{
    "regional_disruptor_specifications": [
        {
            "region_code": "europe-west2",
            "is_whole_region": false,
            "pods": ["cockroachdb-bgwzn"]
        }
    ]
}
```

1. Update the the region code this can be found in the cluster `Overview` screen in the Cloud Console in the `Configuration` section under `Region`. The pod names can be found in the `DB Console`, on the left had menu under `Monitoring` click `Tools`. Click on `Open DB Console`, you will need to have to have `IP Allow List` and a `SQL User` created. Once the DB Console is opened click on `Overview` on the left hand menu. In the node list select a node to isolate from the cluster. You will need the first part of the DNS name, for example `cockroachdb-bgwzn` from the full name of `cockroachdb-bgwzn.cockroachdb.europe-west2.svc.cluster.pcw.gcp-europe-west2`.

1. Update the file `disruption_single_node.json` with this information that has been gathered.

1. The cluster ID needs to be collected from the Cockroach cloud URL for the cluster we are testing. Ensure the cluster is selected in the browser the from the address bar grab the cluster ID from the URL.

Example URL
```
https://cockroachlabs.cloud/cluster/224cf043-016c-4fd9-a358-xxxxxxxxxxx/overview
```

1. The part of the URL we need is `224cf043-016c-4fd9-a358-xxxxxxxxxxx` this is the Cluster ID. Will will use this in the API calls we make to test the cluster. Add this Cluster ID to an environment variable.

```
export cluster=224cf043-016c-4fd9-a358-xxxxxxxxxxx
```

1. Now use the `curl` command to run the first API call to isolate a single node that was described in the `json` file.

```
curl --request PUT \
--url "https://cockroachlabs.cloud/api/v1/clusters/$cluster/disrupt" \
--header "Authorization: Bearer $TOKEN" \
--data-binary '@./disruption_single_node.json'
```

1. Refresh the DB Console and you should see a single node described as `Suspect` in the console. You may need to close and reopen the DB Console as you may have been connected to pod you have isolated.



1. Perform the application testing that you want to perform and ensure you still have access to the database. If this testing takes more than 5 minutes you will see the node mark as dead.

1. Once you are happy you can remove the isolation of the node by running the `curl` command below.

```
curl --request PUT \
--url "https://cockroachlabs.cloud/api/v1/clusters/$cluster/disrupt" \
--header "Authorization: Bearer $TOKEN"
```

1. You may want to continue the application test during this process to see how the application behaves. If you refresh the DB Console you should see the node added back into the cluster and the Under Replicated Ranges return to zero.



