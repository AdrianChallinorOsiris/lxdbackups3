# lxdbackups3
Backup an LXD container to Amazon S3 using S3CMD from https://s3tools.org/s3cmd

## Based on
This work was based on and owes a lot to the work done by Rosco Nap - cloudrkt.com/lxdbackup-script.html

## Description

This is a shell script that will backup the container in to a dated, gzipped file, and then move this to 
Amazon S3. The use of S3 buckets allows the usage of S3 retention policies. For example, the author does 
* Daily backup to a bucket that has a 45 day retention period 
* Monthly backup that is kept online for 90 days then moved the Glacier and retained there for 465 days 

## Prerequisites:

* Install S3CMD (see above) 
* Make sure S3CMD works 
* Create your S3 bucket(s). This is NOT done for you. Use the Amazon S3 Console 
* Set up the retention and migration policies for your bucket(s)

# Installation instructions

Download the script from GIT and save it inyour favorite bin directory. Ensure that you grant the script execute 
privilege. 

```
git clone https://github.com/AdrianChallinorOsiris/lxdbackups3
cd lxdbackups3
chmod +x lxdbackups3
cp lxdbackups3 /usr/bin/
```
The installation and configuration of S3CMD are covered in the documentation of that utility. 

# Testing 

First test that S3CMD is working properly. We do ths by doing the S3 equivalent of ls

```
s3cmd ls s3://<BUCKET> 
```
If everything is as it should be, then you will get back an empty list. Next we test that the LXD Backup works 
by runing it interactively 

```
lxdbackups3 <CONTAINER> <BUCKET> 
```

Expect this to take about 10-15 minutes, depending on the size of the container, the speed of your processor and
your network speed. If all goes well, repeat the ls command above and you should now see your backup file. 

## How do I set it automatically?

You can always run this backup on demand. But if you want to run it via cron, then edit your crontab (crontab -e) 
and add a line such as this. This will do a backup every day at 

```
10 1 * * * flock -n /tmp/lxdbackup.lock lxdbackups3 <CONTAINER> <DAILY-BUCKET>
10 2 1 * * flock -n /tmp/lxdbackup.lock lxdbackups3 <CONTAINER> <MONTHLY-BUCKET>
```



## How do I restore a container?

Download your container image from S3 and import it with LXC or even import. To download use the full bucket and file name

```
s3cmd get s3://<bucket>/<filename> 
lxc image import <filename> --alias <name>
```






