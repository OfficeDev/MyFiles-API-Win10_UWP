---
topic: sample
products:
- OneDrive
- Office 365
languages:
- C#
extensions:
  contentType: samples
  technologies:
  - Azure AD
  createdDate: 8/6/2015 9:07:23 PM
---
# MyFiles-API-Win10-UWP
This repository contains a Universal Windows Platform (UWP) application for Windows 10 that connects to the Office 365 APIs for MyFiles using the new WebAccountManager framework. It displays only folders and images (.png, .jpg, .gif) from OneDrive for Business, so be sure to upload a few before testing.
## Environment Setup ##
The solution was built in Visual Studio 2015 with the Windows Software Development Kit (SDK) for Windows 10.

Office 365 applications are secured by Azure Active Directory, which comes as part of an Office 365 subscription. If you do not have an Office 365 Subscription or associated it with Azure AD, then you should follow the steps to [Set up your Office 365 development environment](https://msdn.microsoft.com/office/office365/HowTo/setup-development-environment "Set up your Office 365 development environment") from MSDN.

## Registering the App ##
When you open the solution in Visual Studio 2015, the application will need to be registered for your tenant. Simply right-click the project and select **Add** > **Connected Service**. Use the connected service wizard to register the application with Azure AD with permissions to read/write MyFiles.

## Using Windows 10's new WebAccountProvider ##
This sample uses Windows 10's new WebAccountProvider instead of a traditional WebAuthenticationBroker that the Azure AD Authentication Libraries (ADAL) have used in the past. The sample below shows how to get access tokens with this new framework. Notice we try to get the token silently at first and then with a forced prompt if it needs user intervention:

    private static async Task<string> GetAccessTokenForResource(string resource)
    {
        string token = null;

        //first try to get the token silently
        WebAccountProvider aadAccountProvider = await WebAuthenticationCoreManager.FindAccountProviderAsync("https://login.windows.net");
        WebTokenRequest webTokenRequest = new WebTokenRequest(aadAccountProvider, String.Empty, App.Current.Resources["ida:ClientID"].ToString(), WebTokenRequestPromptType.Default);
        webTokenRequest.Properties.Add("authority", "https://login.windows.net");
        webTokenRequest.Properties.Add("resource", resource);
        WebTokenRequestResult webTokenRequestResult = await WebAuthenticationCoreManager.GetTokenSilentlyAsync(webTokenRequest);
        if (webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.Success)
        {
            WebTokenResponse webTokenResponse = webTokenRequestResult.ResponseData[0];
            token = webTokenResponse.Token;
        }
        else if (webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.UserInteractionRequired)
        {
            //get token through prompt
            webTokenRequest = new WebTokenRequest(aadAccountProvider, String.Empty, App.Current.Resources["ida:ClientID"].ToString(), WebTokenRequestPromptType.ForceAuthentication);
            webTokenRequest.Properties.Add("authority", "https://login.windows.net");
            webTokenRequest.Properties.Add("resource", resource);
            webTokenRequestResult = await WebAuthenticationCoreManager.RequestTokenAsync(webTokenRequest);
            if (webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.Success)
            {
                WebTokenResponse webTokenResponse = webTokenRequestResult.ResponseData[0];
                token = webTokenResponse.Token;
            }
        }

        return token;
    }

This sample also uses the Discovery Service when establishing a SharePointClient object as seen below. The Unified API avoid this, but isn't yet built into the Visual Studio tooling (you must go into Azure Management Portal):

    private static async Task<SharePointClient> EnsureClient()
    {
        DiscoveryClient discoveryClient = new DiscoveryClient(
                async () => await GetAccessTokenForResource("https://api.office.com/discovery/"));

        // Get the "MyFiles" capability.
        CapabilityDiscoveryResult result = await discoveryClient.DiscoverCapabilityAsync("MyFiles");
            
        return new SharePointClient(result.ServiceEndpointUri, async () => {
            return await GetAccessTokenForResource(result.ServiceResourceId);
        });
    }

## Running the Application ##
The application is built as a Windows 10 UWP application, meaning it can run on both Windows 10 Desktop and Mobile. To debug on a specific device or emulator, simply select the desired option from the debug targets dropdown:

![Local Machine selected in the Windows 10 UWP Debug Targets dropdown](http://i.imgur.com/olh0QBl.png) 

## Windows 10 Desktop: ##
![example of application runing on Windows 10 Desktop displaying a list of folder names and image names](http://i.imgur.com/epMgXsC.png)

## Windows 10 Mobile: ##
![example of application runing on Windows 10 Mobile displaying a list of folder names and image names](http://i.imgur.com/wl5wToG.png)


This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
