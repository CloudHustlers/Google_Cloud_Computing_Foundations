# GSP192
[![](https://github.com/CodingWithHardik/CodingWithHardik/blob/main/img/subscribe_button.png)](https://www.youtube.com/@CloudHustlers)
## Run in cloudshell
The region can be found in the Task 3 point 3
```cmd
export REGION=
```
```cmd
gcloud services disable dataflow.googleapis.com
gcloud services enable dataflow.googleapis.com
```
```cmd
bq mk taxirides
bq mk \
--time_partitioning_field timestamp \
--schema ride_id:string,point_idx:integer,latitude:float,longitude:float,\
timestamp:timestamp,meter_reading:float,meter_increment:float,ride_status:string,\
passenger_count:integer -t taxirides.realtime
gsutil mb gs://$DEVSHELL_PROJECT_ID/
sleep 45
```
```cmd
gcloud dataflow jobs run iotflow \
--gcs-location gs://dataflow-templates/latest/PubSub_to_BigQuery \
--region $REGION \
--staging-location gs://$DEVSHELL_PROJECT_ID/temp \
--parameters inputTopic=projects/pubsub-public-data/topics/taxirides-realtime,outputTableSpec=$DEVSHELL_PROJECT_ID:taxirides.realtime
```
