# Running Apache Spark jobs on Cloud Dataproc

In this lab you will learn how to migrate Apache Spark code to Cloud Dataproc. You will follow a sequence of steps progressively moving more of the job components over to GCP services:
* Run original Spark code on Cloud Dataproc (Lift and Shift)
* Replace HDFS with Cloud Storage (cloud-native)
* Automate everything so it runs on job-specific clusters (cloud-optimized)

## Create Dataproc Cluster
1. In the GCP Console, on the Navigation menu, in the Big Data section, click Dataproc.
2. Click Create Cluster.
3. Enter `sparktodp` for Cluster Name.
4. In the Versioning section, click Change and select 2.0 (Debian 10, Hadoop 3.2, Spark 3.1).

   This version includes Python3 which is required for the sample code used in this lab.

   ![picture alt](https://cdn.qwiklabs.com/NzykmIfgxLXbXUq1pL%2BOlqVGuaoP1pk9zV%2Fo14Y8LYE%3D)

5. Click Select.
6. In the Components > Component gateway section, select Enable component gateway.
7. Under Optional components, Select Jupyter Notebook.
8. Click Create.

## Clone the source repository for the lab
To clone the Git repository for the lab enter the following command in Cloud Shell:
```bash
git -C ~ clone https://github.com/GoogleCloudPlatform/training-data-analyst
```

To locate the default Cloud Storage bucket used by Cloud Dataproc enter the following command in Cloud Shell:
```bash
export DP_STORAGE="gs://$(gcloud dataproc clusters describe sparktodp --region=us-central1 --format=json | jq -r '.config.configBucket')"
```

To copy the sample notebooks into the Jupyter working folder enter the following command in Cloud Shell:
```bash
gsutil -m cp ~/training-data-analyst/quests/sparktobq/*.ipynb $DP_STORAGE/notebooks/jupyter
```

## Log in to the Jupyter Notebook
As soon as the cluster has fully started up you can connect to the Web interfaces. Click the refresh button to check as it may be deployed fully by the time you reach this stage.

1. On the Dataproc Clusters page wait for the cluster to finish starting and then click the name of your cluster to open the **Cluster details** page.
2. Click **Web Interfaces**.
3. Click the **Jupyter** link to open a new Jupyter tab in your browser. 

This opens the Jupyter home page. Here you can see the contents of the /notebooks/jupyter directory in Cloud Storage that now includes the sample Jupyter notebooks used in this lab.

4. Under the **Files** tab, click the **GCS** folder and then click **01_spark.ipynb** notebook to open it.
5. Click **Cell** and then **Run All** to run all of the cells in the notebook.
6. Page back up to the top of the notebook and follow as the notebook completes runs each cell and outputs the results below them.

![picture alt](https://github.com/salenaket/gcp-data-engineering-lab/blob/master/Labs/08-Lab-Running-Apache-Spark-Jobs-on-Dataproc/jupiter-notebook.png)

## Separate Compute and Storage
### Modify Spark jobs to use Cloud Storage instead of HDFS
You start by using the cloud shell to place a copy of the source data in a new Cloud Storage bucket.

1. In the Cloud Shell create a new storage bucket for your source data.

```bash
export PROJECT_ID=$(gcloud info --format='value(config.project)')
gsutil mb gs://$PROJECT_ID
```

2. In the Cloud Shell copy the source data into the bucket.
```bash
wget https://archive.ics.uci.edu/ml/machine-learning-databases/kddcup99-mld/kddcup.data_10_percent.gz
gsutil cp kddcup.data_10_percent.gz gs://$PROJECT_ID/
```
3. Switch back to the `01_spark` Jupyter Notebook tab in your browser.
4. Click **File** and then select **Make a Copy**.
5. When the copy opens, click the `01_spark-Copy1` title and rename it to `De-couple-storage`.
6. Open the Jupyter tab for `01_spark`.
7. Click **File** and then **Save and checkpoint** to save the notebook.
8. Click **File** and then **Close and Halt** to shutdown the notebook.

If you are prompted to confirm that you want to close the notebook click Leave or Cancel.

9. Switch back to the `De-couple-storage` Jupyter Notebook tab in your browser, if necessary.

You no longer need the cells that download and copy the data onto the cluster's internal HDFS file system so you will remove those first.

To delete a cell, you click in the cell to select it and then click the **cut selected cells** icon (the scissors) on the notebook toolbar.

10. Delete the initial comment cells and the first three code cells ( `In [1]`, `In [2]`, and `In [3]`) so that the notebook now starts with the section **Reading in Data**.

11. Replace the contents of cell `In [4]` with the following code. The only change here is create a variable to store a Cloud Storage bucket name and then to point the data_file to the bucket we used to store the source data on Cloud Storage.
```bash
from pyspark.sql import SparkSession, SQLContext, Row
gcs_bucket='[Your-Bucket-Name]'
spark = SparkSession.builder.appName("kdd").getOrCreate()
sc = spark.sparkContext
data_file = "gs://"+gcs_bucket+"//kddcup.data_10_percent.gz"
raw_rdd = sc.textFile(data_file).cache()
raw_rdd.take(5)
```

12. Click **Cell** and then **Run All** to run all of the cells in the notebook.

You will see exactly the same output as you did when the file was loaded and run from internal cluster storage. Moving the source data files to Cloud Storage only requires that you repoint your storage source reference from hdfs:// to gs://.

## Deploy Spark Jobs
1. In the `De-couple-storage` Jupyter Notebook menu, click **File** and select **Make a Copy**.
2. When the copy opens, click the `De-couple-storage-Copy1` and rename it to `PySpark-analysis-file`.
3. Open the Jupyter tab for `De-couple-storage`.
4. Click **File** and then **Save and checkpoint** to save the notebook.
5. Click **File** and then **Close and Halt** to shutdown the notebook.

If you are prompted to confirm that you want to close the notebook click Leave or Cancel.

6. Switch back to the `PySpark-analysis-file` Jupyter Notebook tab in your browser, if necessary.
7. Click the first cell at the top of the notebook.
8. Click **Insert** and select **Insert Cell Above**.
9. Paste the following library import and parameter handling code into this new first code cell:
```bash
%%writefile spark_analysis.py
import matplotlib
matplotlib.use('agg')
import argparse
parser = argparse.ArgumentParser()
parser.add_argument("--bucket", help="bucket for input and output")
args = parser.parse_args()
BUCKET = args.bucket
```

The `%%writefile spark_analysis.py` Jupyter magic command creates a new output file to contain your standalone python script. You will add a variation of this to the remaining cells to append the contents of each cell to the standalone script file.

This code also imports the matplotlib module and explicitly sets the default plotting backend via `matplotlib.use('agg')` so that the plotting code runs outside of a Jupyter notebook.

10. For the remaining cells insert `%%writefile -a spark_analysis.py` at the start of each Python code cell. These are the five cells labelled In [x].
```bash
%%writefile -a spark_analysis.py
```
11. In the last cell, where the Pandas bar chart is plotted remove the `%matplotlib inline` magic command.
12. Make sure you have selected the last code cell in the notebook then, in the menu bar, click **Insert** and select **Insert Cell Below**.
13. Paste the following code into the new cell.
```bash
%%writefile -a spark_analysis.py
ax[0].get_figure().savefig('report.png');
```
14. Add another new cell at the end of the notebook and paste in the following:
```bash
%%writefile -a spark_analysis.py
import google.cloud.storage as gcs
bucket = gcs.Client().get_bucket(BUCKET)
for blob in bucket.list_blobs(prefix='sparktodp/'):
    blob.delete()
bucket.blob('sparktodp/report.png').upload_from_filename('report.png')
```
15. Add a new cell at the end of the notebook and paste in the following:
```bash
%%writefile -a spark_analysis.py
connections_by_protocol.write.format("csv").mode("overwrite").save(
    "gs://{}/sparktodp/connections_by_protocol".format(BUCKET))
```

## Test Automation
1. In the PySpark-analysis-file notebook add a new cell at the end of the notebook and paste in the following:
```bash
BUCKET_list = !gcloud info --format='value(config.project)'
BUCKET=BUCKET_list[0]
print('Writing to {}'.format(BUCKET))
!/opt/conda/miniconda3/bin/python spark_analysis.py --bucket=$BUCKET
```
2. Add a new cell at the end of the notebook and paste in the following:
```bash
!gsutil ls gs://$BUCKET/sparktodp/**
```
3. To save a copy of the Python file to persistent storage, add a new cell and paste in the following:
```bash
!gsutil cp spark_analysis.py gs://$BUCKET/sparktodp/spark_analysis.py
```
4. Click **Cell** and then **Run All** to run all of the cells in the notebook.

## Run the Analysis Job from Cloud Shell.
1. Switch back to your Cloud Shell and copy the Python script from Cloud Storage so you can run it as a Cloud Dataproc Job.
```bash
gsutil cp gs://$PROJECT_ID/sparktodp/spark_analysis.py spark_analysis.py
```
2. Create a launch script.
```bash
nano submit_onejob.sh
```
3. Paste the following into the script:
```bash
#!/bin/bash
gcloud dataproc jobs submit pyspark \
       --cluster sparktodp \
       --region us-central1 \
       spark_analysis.py \
       -- --bucket=$1
```
4. Make the script executable:
```bash
#!/bin/bash
chmod +x submit_onejob.sh
```
5. Launch the PySpark Analysis job:
```bash
#!/bin/bash
./submit_onejob.sh $PROJECT_ID
```
7. In the Cloud Console tab navigate to the **Dataproc > Clusters** page if it is not already open.
8. Click **Jobs**.
9. Click the name of the job that is listed. You can monitor progress here as well as from the Cloud shell. Wait for the Job to complete successfully.
10. Navigate to your storage bucket and note that the output report, `/sparktodp/report.png` has an updated time-stamp indicating that the stand-alone job has completed successfully.
11. The storage bucket used by this Job for input and output data storage is the bucket that is used just the Project ID as the name.
12. Navigate back to the **Dataproc > Clusters** page.
13. Select the `sparktodp` cluster and click Delete. You don't need it any more.
14. Click **CONFIRM**.
15. Close the Jupyter tabs in your browser.