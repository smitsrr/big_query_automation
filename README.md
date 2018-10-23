# BigQuery Automation

We recently automated uploads from our data warehouse to Google Big Query. 

The purpose of this was to have public-facing Tableau workbooks automatically pulling data. Most would just point their tableau server at their database, embed credentials, and schedule a refresh. However, our Tableau server is shared and so we could not establish a secure connection directly to our databases. However, we felt comfortable uploading aggregated data that is public-facing ready to BigQuery and then point Tableau to the BigQuery-hosted tables. 

We use Wherescape Red to manage our scheduled ETL procedures, and fortunately it can also run custom powershell scripts. Google has created some Powershell modules to conveniently make API calls for bulk data uploads. 

## Setup

**1. [cloud.google.com]**
  * You will need to create an account (or use any gmail account), within which you will need a Project and a Dataset. 
  * I highly recommend creating a service account since this user will be granted access to google assets on potentially several machines (and schedulers). Likely more than one individual will also need to know the password for this account, and a service account means that the account is independent of a single individual. 

**2. [Google Cloud SDK](https://cloud.google.com/sdk/)**
  * Every machine that runs the powershell script needs to have the SDK installed. 
  * Execute after downloading to install (choosing defaults), and authenticate using your service account. 
  * This step saves authenticated accounts on your machine. 
  * To confirm your account was registered and authenticated run `gcloud auth list` in powershell. 
  * Also in powershell, run `get-module -listAvailable` to and make sure that GoogleCloud and GoogleCloudBeta are available. 
  * If you believe the module was installed, but powershell is not seeing it, run `$Env:PSMODULEPATH` to see where powershell is looking for installed modules. 
  * Note: I believe the modules are installed when you install the SDK, but I can't recall exactly. 

3. Project Confirmation
  * The SDK stores project configurations available to the machine. 
  * In your terminal run `gcloud init` to set your project preferences. 





