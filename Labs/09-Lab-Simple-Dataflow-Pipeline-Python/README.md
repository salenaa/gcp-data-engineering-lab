# A Simple Dataflow Pipeline (Python) 2.5

In this lab, you will open a Dataflow project, use pipeline filtering, and execute the pipeline locally and on the cloud.
* Open Dataflow project
* Pipeline filtering
* Execute the pipeline locally and on the cloud

## Download Code Repository
**Note:** Compute VM instance need to be created.

Download a code repository for use in this lab. In the **training-vm** SSH terminal enter the following:
```bash
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```

## Create a Cloud Storage bucket
1. In the Console, on the Navigation menu, click Home.
2. Select and copy the Project ID.

For simplicity you will use the Qwiklabs Project ID, which is already globally unique, as the bucket name.

3. In the Console, on the Navigation menu, click **Cloud Storage > Browser**.
4. Click Create Bucket.

Specify the following, and leave the remaining settings as their defaults:

Property      | Value (type value or select option as specified)
------------- | ----------------------------------------------------
Name          | <your unique bucket name (Project ID)>
Location type | Multi-Region
Location      | <Your location>

5. In the **training-vm** SSH terminal enter the following to create an environment variable named "BUCKET" and verify that it exists with the echo command.
```bash
BUCKET="<your unique bucket name (Project ID)>"
echo $BUCKET
```

## Pipeline filtering
Return to the **training-vm* SSH terminal and navigate to the directory `training-data-analyst/courses/data_analysis/lab2/python` and view the file `grep.py`.

View the file with Nano. Do not make any changes to the code. Press Ctrl+X to exit Nano.
```bash
cd ~/training-data-analyst/courses/data_analysis/lab2/python
nano grep.py
```

## Execute the pipeline locally
1. In the **training-vm** SSH terminal, locally execute `grep.py`.
```bash
python3 grep.py
Copied!
```

The output file will be `output.txt`. If the output is large enough, it will be sharded into separate parts with names like: `output-00000-of-00001`.

2. Locate the correct file by examining the file's time.
```bash
ls -al /tmp
Copied!
```
3. Examine the output file(s).
4. You can replace "-*" below with the appropriate suffix.
```bash
cat /tmp/output-*
```

## Execute the pipeline on the cloud
1. Copy some Java files to the cloud. In the training-vm SSH terminal, enter the following commmand:
```bash
gsutil cp ../javahelp/src/main/java/com/google/cloud/training/dataanalyst/javahelp/*.java gs://$BUCKET/javahelp
```
2. Using Nano, edit the Dataflow pipeline in `grepc.py`.
```bash
nano grepc.py
```
3. Replace **PROJECT** and **BUCKET** with your Project ID and Bucket name.
4. Submit the Dataflow job to the cloud:
```bash
python3 grepc.py
```
**Note:** You may ignore the message: WARNING:root:Make sure that locally built Python SDK docker image has Python 3.7 interpreter. Your Dataflow job will start successfully.
Because this is such a small job, running on the cloud will take significantly longer than running it locally (on the order of 7-10 minutes).

5. Return to the browser tab for Console.
6. On the Navigation menu, click **Dataflow** and click on your job to monitor progress.
![picture](https://cdn.qwiklabs.com/PJjekfYHBQcxU4aMa%2Fej7C5KoXXXUuSYZdkn2QtU69w%3D)
7. Wait for the job status to turn to **Succeeded**.
8. Examine the output in the Cloud Storage bucket.
9. On the Navigation menu, click **Cloud Storage > Browser** and click on your bucket.
10. Click the **javahelp** directory.

This job will generate the file output.txt. If the file is large enough it will be sharded into multiple parts with names like: `output-0000x-of-000y`. You can identify the most recent file by name or by the Last modified field.

11. Click on the file to view it.
12. Alternatively, you can download the file via the training-vm SSH terminal and view it:
```bash
gsutil cp gs://$BUCKET/javahelp/output* .
cat output*
```