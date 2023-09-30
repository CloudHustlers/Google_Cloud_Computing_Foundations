# GSP315 
[![](https://github.com/CodingWithHardik/CodingWithHardik/blob/main/img/subscribe_button.png)](https://www.youtube.com/@CloudHustlers)
## Run in cloudshell
```cmd
export USERNAME2=
```
```cmd
export BUCKET_NAME=
```
```cmd
export TOPIC_NAME=
```
```cmd
export FUNCTION_NAME=
```
```cmd
export REGION=${ZONE::-2}
gsutil mb -l $REGION gs://$DEVSHELL_PROJECT_ID-bucket
gcloud pubsub topics create $TOPIC_NAME
mkdir cloudhustlers
cd cloudhustlers
cat > index.js << EOF
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const { Storage } = require('@google-cloud/storage');
const gcs = new Storage();
const { PubSub } = require('@google-cloud/pubsub');
const imagemagick = require("imagemagick-stream");
exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "$TOPIC_NAME";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log("Processing Original: gs://"+bucketName+"/"+fileName);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log("Error: "+err);
            reject(err);
          })
          .on("finish", () => {
            console.log("Success: "+fileName+" → "+newFilename);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log("Message "+messageId+" published.");
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });
          });
      });
    }
    else {
      console.log("gs://"+bucketName+"/"+fileName+" is not an image I can handle");
    }
  }
  else {
    console.log("gs://"+bucketName+"/"+fileName+" already has a thumbnail");
  }
};
EOF
cat > package.json << EOF
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/pubsub": "^2.0.0",
    "@google-cloud/storage": "^5.0.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
EOF
gcloud functions deploy $FUNCTION_NAME \
--runtime nodejs14 \
--trigger-resource $DEVSHELL_PROJECT_ID-bucket \
--trigger-event google.storage.object.finalize \
--entry-point thumbnail \
--region=$REGION
curl -o map.jpg https://storage.googleapis.com/cloud-training/gsp315/map.jpg
gsutil cp map.jpg gs://$DEVSHELL_PROJECT_ID-bucket/map.jpg
gcloud projects remove-iam-policy-binding $DEVSHELL_PROJECT_ID \
--member=user:$USERNAME2 \
--role=roles/viewer
```
