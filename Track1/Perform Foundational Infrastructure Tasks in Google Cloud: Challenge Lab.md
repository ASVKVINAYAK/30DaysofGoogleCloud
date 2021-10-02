# GSP315: Perform Foundational Infrastructure Tasks in Google Cloud: Challenge Lab

## Task 1: Create a bucket

- **Navigation menu > CLOUD STORAGE > Browser > Create Bucket**
- **Name your bucket > Enter GCP Project ID > Continue**
- **Choose where to store your data > Region: us-east1 > Continue**
- **Use default for the remaining**
- **Create**

## Task 2: Create a Pub/Sub topic

- **Navigation menu > BIG DATA > Pub/Sub**
- **Create Topic > Name: MyTopic> Create Topic**

## Task 3: Create the thumbnail Cloud Function

- **Navigation menu > COMPUTE > Cloud Functions > Create Function**
- **FunctionName: cloudfunction**
- **Region: us-east1** 
- **Trigger: Cloud Storage**
- **Event type: Finalize/Create** 
- **Bucket: BROWSE > Select the qwiklabs bucket**
- **Next**

 - **In Entry point: thumbnail**
 - **In line 16 of index.js replace the text REPLACE_WITH_YOUR_TOPIC ID with the Topic ID you created in task 2.**
 
 **index.js**

 ```bash
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
  const topicName = "REPLACE_WITH_YOUR_TOPIC ID";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
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
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
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
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });
          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```
**package.json**
```bash
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/functions-framework": "^1.1.1",
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
```
- **Download the image from URL:** https://storage.googleapis.com/cloud-training/gsp315/map.jpg
- **Navigation menu > STORAGE > Storage > Select your bucket > Upload files**
- **Refresh bucket**

## Task 4: Remove the previous cloud engineer
- **Navigation menu > IAM & Admin**
- **Search for the "Username 2" > Edit > Delete Role**

**This Lab is Done** :blush:





