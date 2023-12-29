# cURL Request Generator

Generate cURL requests from OpenAPI specifications v2.0 and v3.0

## Usage

```pwsh
USAGE:
    curlgenerator [URL or input file] [OPTIONS]

EXAMPLES:
    curlgenerator ./openapi.json
    curlgenerator ./openapi.json --output ./
    curlgenerator ./openapi.json --output-type onefile
    curlgenerator https://petstore.swagger.io/v2/swagger.json
    curlgenerator https://petstore3.swagger.io/api/v3/openapi.json --base-url https://petstore3.swagger.io
    curlgenerator ./openapi.json --authorization-header Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
    curlgenerator ./openapi.json --azure-scope [Some Application ID URI]/.default

ARGUMENTS:
    [URL or input file]    URL or file path to OpenAPI Specification file

OPTIONS:
                                           DEFAULT                                                                                                                            
    -h, --help                                                  Prints help information                                                                                       
    -v, --version                                               Prints version information                                                                                    
    -o, --output <OUTPUT>                  ./                   Output directory                                                                                              
        --no-logging                                            Don't log errors or collect telemetry                                                                         
        --skip-validation                                       Skip validation of OpenAPI Specification file                                                                 
        --authorization-header <HEADER>                         Authorization header to use for all requests                                                                  
        --content-type <CONTENT-TYPE>      application/json     Default Content-Type header to use for all requests                                                           
        --base-url <BASE-URL>                                   Default Base URL to use for all requests. Use this if the OpenAPI spec doesn't explicitly specify a server URL
        --output-type <OUTPUT-TYPE>        OneRequestPerFile    OneRequestPerFile generates one script file per cURL request. OneFile generates a single script file for all cURL requests
        --azure-scope <SCOPE>                                   Azure Entra ID Scope to use for retrieving Access Token for Authorization header                              
        --azure-tenant-id <TENANT-ID>                           Azure Entra ID Tenant ID to use for retrieving Access Token for Authorization header                          
```

Running the following:

```sh
curlgenerator https://petstore.swagger.io/v2/swagger.json
```

Outputs the following:

```sh
cURL Request Generator v0.1.1
Support key: mbmbqvd

OpenAPI statistics:
 - Path Items: 14
 - Operations: 20
 - Parameters: 14
 - Request Bodies: 9
 - Responses: 20
 - Links: 0
 - Callbacks: 0
 - Schemas: 67

Files: 20
Duration: 00:00:02.3089450
```

Which will produce the following files:

```sh
-rw-r--r-- 1 christian 197121  593 Dec 10 10:44 DeleteOrder.ps1        
-rw-r--r-- 1 christian 197121  231 Dec 10 10:44 DeletePet.ps1
-rw-r--r-- 1 christian 197121  358 Dec 10 10:44 DeleteUser.ps1
-rw-r--r-- 1 christian 197121  432 Dec 10 10:44 GetFindPetsByStatus.ps1
-rw-r--r-- 1 christian 197121  504 Dec 10 10:44 GetFindPetsByTags.ps1  
-rw-r--r-- 1 christian 197121  371 Dec 10 10:44 GetInventory.ps1       
-rw-r--r-- 1 christian 197121  247 Dec 10 10:44 GetLoginUser.ps1       
-rw-r--r-- 1 christian 197121  291 Dec 10 10:44 GetLogoutUser.ps1      
-rw-r--r-- 1 christian 197121  540 Dec 10 10:44 GetOrderById.ps1
-rw-r--r-- 1 christian 197121  275 Dec 10 10:44 GetPetById.ps1
-rw-r--r-- 1 christian 197121  245 Dec 10 10:44 GetUserByName.ps1
-rw-r--r-- 1 christian 197121  513 Dec 10 10:44 PostAddPet.ps1
-rw-r--r-- 1 christian 197121  521 Dec 10 10:44 PostCreateUser.ps1
-rw-r--r-- 1 christian 197121  610 Dec 10 10:44 PostCreateUsersWithListInput.ps1
-rw-r--r-- 1 christian 197121  464 Dec 10 10:44 PostPlaceOrder.ps1
-rw-r--r-- 1 christian 197121  299 Dec 10 10:44 PostUpdatePetWithForm.ps1
-rw-r--r-- 1 christian 197121  274 Dec 10 10:44 PostUploadFile.ps1
-rw-r--r-- 1 christian 197121  513 Dec 10 10:44 PutUpdatePet.ps1
-rw-r--r-- 1 christian 197121  541 Dec 10 10:44 PutUpdateUser.ps1
```

In this example, the contents of `PostAddPet.ps1` looks like this:

```pwsh
curl -X 'POST' 'https://petstore3.swagger.io/api/v3/pet' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9' \
  -H 'Content-Type: application/json' \
  -d '{
  "id": 10,
  "name": "doggie",
  "category": {
    "id": 1,
    "name": "Dogs"
  },
  "photoUrls": [
    "string"
  ],
  "tags": [
    {
      "id": 0,
      "name": "string"
    }
  ],
  "status": "available"
}'
```

Here's an advanced example of generating cURL requests for a REST API hosted on Microsoft Azure that uses the Microsoft Entra ID service as an STS. For this example, I use PowerShell and Azure CLI to retrieve an access token for the user I'm currently logged in with.

```powershell
az account get-access-token --scope [Some Application ID URI]/.default `
| ConvertFrom-Json `
| %{
    curlgenerator `
        https://api.example.com/swagger/v1/swagger.json `
        --authorization-header ("Bearer " + $_.accessToken) `
        --base-url https://api.example.com `
        --output ./HttpFiles 
}
```

You can also use the `--azure-scope` and `azure-tenant-id` arguments internally use `DefaultAzureCredentials` from the `Microsoft.Extensions.Azure` NuGet package to retrieve an access token for the specified `scope`.

```powershell
curlgenerator `
  https://api.example.com/swagger/v1/swagger.json `
  --azure-scope [Some Application ID URI]/.default `
  --base-url https://api.example.com `
  --output ./HttpFiles 
```

#

For tips and tricks on software development, check out [my blog](https://christianhelle.com)

If you find this useful and feel a bit generous then feel free to [buy me a coffee ☕](https://www.buymeacoffee.com/christianhelle)