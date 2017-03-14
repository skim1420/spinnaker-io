First, you will need to create an Azure Active Directory [service principal](https://azure.microsoft.com/en-us/documentation/articles/active-directory-application-objects/) for authentication. You can create a service principal from the Azure Portal or via the command line. This tutorial demonstrates using the Azure Command-Line Interface (Azure CLI). **Important:**  Ensure you are on the latest version of the Azure CLI or at least version  0.10.2.

**Important:** Keep the output values from the commands you execute.  You will use the values when configuring the Azure Driver for Spinnaker.

1.  Install the [Azure CLI](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/) for the platform of your choice. After installing the Azure CLI you can run commands from a command line interface on your platform.

2.  Open the command prompt and type **azure help.** If the command executes, you have successfully installed the Azure CLI.

3.  Type **azure login**. The command outputs a code and URL such as <https://aka.ms/devicelogin>. Follow the instructions on the screen to log in. You should see a successful login. See [connect to Azure subscription from CLI](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-connect/) for more information.

4.  Type **azure config mode arm** to enter [Application Resource Manager mode](https://azure.microsoft.com/en-us/documentation/articles/azure-cli-arm-commands).

5.  Type **azure account list** to obtain the Azure subscription ID. Copy the subscription ID for future steps.

6.  Type **azure account set &lt;enter subscription ID with no angle brackets&gt;** to set the Azure subscription.

7. In the following command replace the uris with your own that identify your application.  Type **azure ad sp create --name "spinnaker" --home-page "http://InsertYourURIHere.com" --identifier-uris "http://InsertYourURIHere.com" --password EnterPasswordHere**

	[See here for more information on the step above](https://azure.microsoft.com/en-us/documentation/articles/resource-group-create-service-principal-portal/)

8.  The command above creates an application and service principal.  The command outputs data for  AppID (also called ClientId) and ObjectId. Copy and keep this data for future steps.  **Note:**  The ClientId value will be listed under the service principal name. The password you entered above will be used later for the AppKey value when configuring the Azure driver for Spinnaker.

9. Type **azure role assignment create &lt;insert Object Id from step above with no angle brackets&gt;** **-o Owner -c /subscriptions/&lt;insert subscription ID from step 5 with no angle brackets here&gt;**

10. Type **azure account show.** Copy the Tenant ID for use on future steps.

11. Verify the service principal login by typing **azure login -u "&lt;insert app id from step 8 without the angle brackets here&gt;" --service-principal --tenant "&lt;insert the tenant id without brackets from step 10 here&gt;"**

