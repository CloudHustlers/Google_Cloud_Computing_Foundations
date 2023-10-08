# GSP001
[![](https://github.com/CodingWithHardik/CodingWithHardik/blob/main/img/subscribe_button.png)](https://www.youtube.com/@CloudHustlers)
## Run in cloudshell
```cmd
export ZONE=Zone
```
```cmd
gcloud compute instances create gcelab \
--zone=$ZONE \
--machine-type=e2-medium \
--image-project=debian-cloud \
--image-family=debian-11 \
--tags=http-server
gcloud compute firewall-rules create allow-http \
--action=ALLOW \
--direction=INGRESS \
--rules=tcp:80 \
--source-ranges=0.0.0.0/0 \
--target-tags=http-server
gcloud compute instances create gcelab2 --machine-type e2-medium --zone=$ZONE
gcloud compute ssh gcelab --zone=$ZONE
```
```cmd
sudo apt-get update
sudo apt-get install -y nginx
ps auwx | grep nginx
exit
```
