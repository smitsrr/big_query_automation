# BigQuery Automation

We recently automated uploads from our data warehouse to Google Big Query. 

The purpose of this was to have public-facing Tableau workbooks automatically pulling data. Most would just point their tableau server at their database, embed credentials, and schedule a refresh. However, our Tableau server is shared and so we could not establish a secure connection directly to our databases. However, we felt comfortable uploading aggregated data that is public-facing ready to BigQuery and then point Tableau to the BigQuery-hosted tables. 

We use Wherescape Red to manage our scheduled ETL procedures, and fortunately it can also run custom powershell scripts. Google has created some Powershell modules to conveniently make API calls for bulk data uploads. 

## Setup

**1. [Google Cloud](cloud.google.com)**
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

**3. Project Confirmation**
  * The SDK stores project configurations available to the machine. 
  * In your terminal run `gcloud init` to set your project preferences. 
  * At this point in your setup you should be able to run a few commands in powershell to make sure you're configured. Both of the following should return some metadata: 
```
Get-BqDataset "[dataset_id]"
Get-BqTable "[dataset_id]" -DatasetId "[table_id]" 
```
  * Get your dataset and table IDs are found in your google cloud interface.
![DatasetID](https://github.com/smitsrr/big_query_automation/blob/master/dataset_id.PNG)
![TableID](https://github.com/smitsrr/big_query_automation/blob/master/table_id.PNG)
  * Note that up to this point you are only authenticated to query the metadata associated with your table, dataset, and project.

**4. Authentication keys to push/query data**
  * Because google charges for data transmission (upload/download), they require an extra authentication request before you are able to run those specific commands.
  * At [](cloud.google.com) go to Console > Go to project settings > Service Accounts > Create Service Account
    * Name: succinct name for the purpose of this account (e.g., "warehouse upload")
    * Description: you’ll thank yourself later if you make this descriptive
    * Permissions: > Choose Project > Editor (at least)
    * Grant users access to this service account (I skipped this step)
    * Create Key - Do this! This produces a JSON file with your credentials. Keep this in a shared, but secured location. This allows the bigQuery powershell module to edit your data, and perform operations that charge your account.

**5. Set Environment Variable**
  * On all machines that will run the powershell scripts, you must have an environment variable that points to the JSON permissions file. Google SDK expects this variable to exist, and uses it to find your JSON authentication file. 
  * Always restart your system after setting environment variables!!
    * Name: GOOGLE_APPLICATION_CREDENTIALS
    * Value: path to the JSON permissions file (including the file name)
  * To make sure the environment variable is set, and available, in Powershell run `$env:GOOGLE_APPLICATION_CREDENTIALS`

**6. Upload**
  * As long as you have created a BigQuery table, you should be able to upload data using powershell.
  * Create a .csv file with the exact specifications (column names/data types) as what you declared in BigQuery.
  * Tell GoogleCloudBeta which table you want to alter: 
```
$table = Get-BqTable "[TABLE_ID]" -DatasetId "[DATASET_ID]"
```
  * With that declared table, truncate the data and write all of the new data. We are flushing and filling the tables daily, but there are also options for just inserting data. You can specify if your file has leading (header) rows with the `-SkipLeadingRows` argument. 
```
$table | Add-BqTableRow CSV "[PATH_TO_FILE]\[FILE_NAME].csv" -SkipLeadingRows 0 -WriteMode WriteTruncate
```
  * If you go back to the table details in BigQuery, you should see that your table was Last Modified at the time you ran that command. 

**7. Closing the loop**
  * To make the entire process automated, our ETL scheduling system creates aggregated tables, exports them to a .csv file, then executes the powershell script that updates the BigQuery tables. 
  * When we have a new table we want uploaded to BigQuery we simple have to create the aggregated tables with associated export in our ETL tool, creat the dataset (if we want it in a separate project) and table in BigQuery, then add two lines of code to our scheduled powershell script. 