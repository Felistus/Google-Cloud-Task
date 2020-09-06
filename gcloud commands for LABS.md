1. # Getting Started with Compute Engine
## create vm-1:
```
gcloud compute instances create "my-vm-1" --zone=us-central1-a --machine-type "n1-standard-1" --image-project "debian-cloud" --image=debian-9-stretch-v20200902 --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=my-vm-1 --tags=http-server --subnet "default"
```
```
gcloud compute --project=jaro-tech-reviews firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
```
## create vm-2:
```
gcloud compute instances create "my-vm-2" --zone=us-central1-b --machine-type "n1-standard-1" --image-project "debian-cloud" --image=debian-9-stretch-v20200902 --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=my-vm-1 --subnet "default"
```
## connect between VM instances
### ssh into vm-2 and ping vm-1
```
ping my-vm-2
```
### ssh into vm-1 and install the Nginx webserver
```
sudo apt-get install nginx-light -y
```
### edit home page of the webserver:
```
sudo nano /var/www/html/index.nginx-debian.html
```
### view edited home page (enter command on vm-1 command prompt)
```
curl http://localhost/
```
### enter command on vm-2 command prompt
```
curl http://my-vm-1/
```
`exit`


2. # Getting Started with Cloud Storage and Cloud SQL
## export my zone
```
export MY_ZONE=us-central1-b
```
## export my location
```
export LOCATION=us
```
## download an editable deployment manager template from a public storage
```
gsutil cp gs://cloud-training/gcpfcoreinfra/mydeploy.yaml mydeploy.yaml
```
## replace default project_id with mine
```
sed -i -e "s/PROJECT_ID/$DEVSHELL_PROJECT_ID/" mydeploy.yaml
```
## replace default zone with mine
```
sed -i -e "s/ZONE/$MY_ZONE/" mydeploy.yaml
```
## view template
```
cat mydeploy.yaml
```
## build deployment from the template
```
gcloud deployment-manager deployments create my-deployment --config mydeploy.yaml
```
## edit the template and add your configuration
```
nano mydeploy.yaml
```
### configs to add:
-  ```
 - name: default-allow-http
   type: compute.v1.firewall
  properties:
    targetTags: ["http"]
    sourceRanges: ["0.0.0.0/0"]
    allowed:
      - IPProtocol: TCP
        ports: ["80"] 
```
- add under the metadata, **value** property
```
apt-get update; apt-get install apache2 php php-mysql -y; service apache2 restart
```
**Ctrl+0 and Enter** to save; **Ctrl+X** to exit
## update my deployment
```
gcloud deployment-manager deployments update my-deployment --config mydeploy.yaml
```
## create a cloud storage bucket
```
gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID
```
## retrieve a banner image from a publicly accessible Cloud Storage location
```
gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png
```
## copy banner image to my bucket
```
gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```
## I modify the Access Control List of the object I just created so that it is readable by everyone
```
gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png
```
## create a second generation Cloud SQL instance
```
gcloud sql instances create blog-db --database-version=MYSQL_8_0 --tier=db-n1-standard-1 --region=us-central1 --root-password=myPassword
```
### create a new user
```
gcloud sql users create blogdbuser --instance=blog-db --host=% --password=myPassword
```
### add a public network
```
gcloud sql instances patch [instance name] --assign-ip
```
```
gcloud sql instances patch [instance] --authorized-networks=<external IP address/32>
```
## configure an application in a Compute Engine instance to use Cloud SQL
- navigate to the VM instance and SSH
- change working directory
```
cd /var/www/html
```
- edit index.php
```
sudo nano index.php
```
- restart server after editing
```
sudo service apache2 restart
```
- test site using vm external IP
*35.226.246.29/index.php*
- reedit index.php and add SQL IP and Password
```
sudo nano index.php
```
- save and restart web server
```
sudo service apache2 restart
```
## configure an application in a Compute Engine instance to use a Cloud Storage object
-  goto cloud storage, copy the public URL of the image and add to the index.php file
- navigate to the VM instance and SSH
- change working directory
```
cd /var/www/html
```
- edit index.php
```
sudo nano index.php
```
- add image url to the file before the `<h1>` tag
`<img src='https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png'>`
- save and restart web server
```
sudo service apache2 restart
```
- test your page again using the VN external IP address/index.php.
eg: 35.192.208.2/index.php



3. # Getting Started with App Engine

### list the active account
```
gcloud auth list
```
### list project ID
```
gcloud config list project
```

##  initialize App Engine
```
gcloud app create --project=$DEVSHELL_PROJECT_ID
```
-  clone the source code repository for a sample application in the hello_world directory
```
git clone https://github.com/GoogleCloudPlatform/python-docs-samples
```
- navigate to the source directory
```
cd python-docs-samples/appengine/standard_python3/hello_world
```

## Run Hello World application locally
- update the packages list
```
sudo apt-get update
```
- setup a python virtual environment
```
sudo apt-get install virtualenv
```
**If prompted [Y/n], press Y and then Enter.**
```
virtualenv -p python3 venv
```
- activate the virtual environment
```
source venv/bin/activate
```
- navigate to your project directory and install dependencies
```
pip install  -r requirements.txt
```
- run the application
```
python main.py
```
- in Cloud Shell, click Web preview (Web Preview) > Preview on port 8080 to preview the application.
- abort test using *Ctrl+C*

## deploy and run Hello World on App Engine
- navigate to the source directory
```
cd ~/python-docs-samples/appengine/standard_python3/hello_world
```
- deploy your Hello World application
```
gcloud app deploy
```
- launch browser
```
gcloud app browse
```
**Disable app engine application**

**END**