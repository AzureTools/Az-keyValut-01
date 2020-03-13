# Tutorial: Use Azure Key Vault with an Azure web app in .NET

https://docs.microsoft.com/en-us/azure/key-vault/tutorial-net-create-vault-azure-web-app

Before you begin, read Key Vault basic concepts.
https://docs.microsoft.com/en-us/azure/key-vault/basic-concepts


az login

## To list locations: az account list-locations --output table
```
az group create --name "<YourResourceGroupName>" --location "West US"

az keyvault create --name "<YourKeyVaultName>" --resource-group "<YourResourceGroupName>" --location "West US"
```

## add secret
```
az keyvault secret set --vault-name "<YourKeyVaultName>" --name "AppSecret" --value "MySecret"

az keyvault secret show --name "AppSecret" --vault-name "<YourKeyVaultName>"
```

Create a .NET Core web app

Install these NuGet packages:

AppAuthentication
https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication

KeyVault
https://www.nuget.org/packages/Microsoft.Azure.KeyVault

https://www.nuget.org/packages/Microsoft.Azure.KeyVault
```
  using Microsoft.Azure.KeyVault;
  using Microsoft.Azure.KeyVault.Models;
  using Microsoft.Azure.Services.AppAuthentication;
 ```
 
 
 Your code in the AboutModel class should look like this:
 ```
 public class AboutModel : PageModel
 {
     public string Message { get; set; }

     public async Task OnGetAsync()
     {
         Message = "Your application description page.";
         int retries = 0;
         bool retry = false;
         try
         {
             /* The next four lines of code show you how to use AppAuthentication library to fetch secrets from your key vault */
             AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider();
             KeyVaultClient keyVaultClient = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));
             var secret = await keyVaultClient.GetSecretAsync("https://<YourKeyVaultName>.vault.azure.net/secrets/AppSecret")
                     .ConfigureAwait(false);
             Message = secret.Value;
         }
         /* If you have throttling errors see this tutorial https://docs.microsoft.com/azure/key-vault/tutorial-net-create-vault-azure-web-app */
         /// <exception cref="KeyVaultErrorException">
         /// Thrown when the operation returned an invalid status code
         /// </exception>
         catch (KeyVaultErrorException keyVaultException)
         {
             Message = keyVaultException.Message;
         }
     }

     // This method implements exponential backoff if there are 429 errors from Azure Key Vault
     private static long getWaitTime(int retryCount)
     {
         long waitTime = ((long)Math.Pow(2, retryCount) * 100L);
         return waitTime;
     }

     // This method fetches a token from Azure Active Directory, which can then be provided to Azure Key Vault to authenticate
     public async Task<string> GetAccessTokenAsync()
     {
         var azureServiceTokenProvider = new AzureServiceTokenProvider();
         string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://vault.azure.net");
         return accessToken;
     }
 }
 
 ```
 
 
# Run the web app
On the main menu of Visual Studio 2019, select Debug > Start, with or without debugging.
In the browser, go to the About page.
The value for AppSecret is displayed.

 
 # Enable a managed identity
 
 
 ```
 az webapp identity assign --name "<YourAppName>" --resource-group "<YourResourceGroupName>"
 ```
 
 Replace <YourAppName> with the name of the published app on Azure.
For example, if your published app name was MyAwesomeapp.azurewebsites.net, replace <YourAppName> with MyAwesomeapp


 
 Make a note of the PrincipalId (When you publish)
 ```
 {
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "SystemAssigned"
}
```

NOTE:
switching the Identity / System assigned setting to On in the web application properties.



 # Assign permissions to your app
 Replace <YourKeyVaultName> with the name of your key vault, and replace <PrincipalId> with the value of the PrincipalId in the following command:
 
 ```
 az keyvault set-policy --name '<YourKeyVaultName>' --object-id <PrincipalId> --secret-permissions get list
 ```
 This command gives the identity (MSI) of the app service permission to do get and list operations on your key vault.
 
 
 
 Azure Key Vault Developer's Guide
 https://docs.microsoft.com/en-us/azure/key-vault/key-vault-developers-guide
 
 
 
 
