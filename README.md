#Running a Spark - Python script on a 3 node AWS EMR cluster (Part 1)

This assignment is based on the 3rd chapter of the book - 'Advanced Big Data Analytics with Spark'. You can find a copy of the book here - https://drive.google.com/file/d/0B5BGwUuGBNtNQzByU0JwX29ycTQ/view.

In this assignment, you will be required to build a recommendation system using Spark and MLib using a dataset published by AudioScrobbler. This data is 500MB uncompressed and can be downloaded here - http://www-etud.iro.umontreal.ca/~bergstrj/audioscrobbler_data.html

You can glance through the chapter and understand how to build the recommendation system. The chapter will walk you through Scala code; however; you are expected to code in Python. You will be running your recommendation system on a 3-node AWS-EMR cluster. You can work in teams of 2 to complete this assignment. The assignment is due 12th December 11:59pm.


##Setup

We will first see how to set up a 3 node [AWS - ELastic Map Reduce cluster with Spark](http://docs.aws.amazon.com/ElasticMapReduce/latest/DeveloperGuide/emr-spark-launch.html) installed on it. To follow along, make sure you have a functioning AWS account and an IAM user created. You may want to refer to this [AWS Documentation](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html) for setting up AWS CLI.

Also, this service offered by AWS is chargeable. Do not keep you cluster running for too long and remember to keep checking your AWS bill under the Billing and Cost Management option.

AWS offers an easy to use dashboard for creating and launching a cluster and submitting a Spark job to it. Begin by clicking on EMR from the list of services offered on your AWS dashboard. Then click on Create Cluster and you will be directed to this page: 

![SetupImage](http://i.imgur.com/arhxIA9.png)


* Give the cluster a name, enable logging(this will be very useful to troubleshoot in case of exceptions) and select 'Cluster' in launch mode. 
* In software configuration, select Spark from the Applications list. 
* If you have not created an EC2 key-pair yet, create one, download it and enter its name here. Select default permissions and then go ahead and click on 'Create Cluster'.

You will now be directed to this page.
![Cluster Details](http://i.imgur.com/7EQguTV.png)

The cluster remains in the 'Starting' state for about 10 - 15 minutes. Once the cluster is ready for use, the status will change to 'Waiting'. The cluster is now ready to use and we can move on to uploading our data to S3.


##Uploading data

On your AWS console, from the Services dropdown on the dashboard, select S3 and you should see a S3 bucket here. It is the same bucket that is going to store logs for your cluster. Click on the S3 bucket, right click and select 'Create Folder'. Give this folder a name (e.g 'audio_data'). Click on the folder name, from the Actions drop down, select upload and assuming you have the audioscrobbler dataset stored locally on your machine, click on 'Add files' and upload these to the folder you just created on the bucket.

![S3 Bucket](http://i.imgur.com/DbzTwy7.png)
##Deploying Code 

You will have to SSH into your master node to copy your python code to this node. After having completed the AWS CLI setup, cd into the folder where you have your key-pair.pem saved. Copy the cluster id or the public DNS of the master node from cluster info and run any one of the following two commands:

* aws emr ssh --cluster-id <id> --key-pair-file <.pem file name>

OR

* ssh hadoop@<public dns of master> --key-pair-file <.pem file name>

Once you have sshed into your master node, create a new folder, say python_code and cd into it. Create a new file here by using the vi command, copy paste your recommendation system code here and save it. The code is now ready to be used by all the nodes in your cluster. Make sure to change the paths to all data files in your code. The paths should now point to the folder you created in the S3 bucket(ed. "s3://bucket_name/audiodata/artist_data.txt").  Let us now see how to submit a job to this cluster. 

##Submitting Job

From the cluster list, select the cluster you just created. Click on 'Add Step'. See the screen below for the options to configure -

![Configure Steps](http://i.imgur.com/cUzMbUt.png)

You are submitting this job using a script-runner provided by AWS. You are passing two arguments to it, the first one being "spark-submit" and the second is the path to python file you created on the master node.

Click on Add and refresh the page to see if the step was successfully added. The cluster state changes from Waiting to Running if the job is accepted and is running successfully. If the job fails, wait for a couple of minutes for the logs to appear in step info:

![Logs](http://i.imgur.com/Q3miV9F.png)

If the status of your step changes to Completed after about 10 minutes, you are done. If the step fails, you can check controller/stderr/stdout logs to troubleshoot. 
