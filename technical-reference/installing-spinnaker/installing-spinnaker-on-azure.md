1.  You will need data from the Azure section on the [Spinnaker Cloud Provider Setup](http://www.spinnaker.io/v1.0/docs/target-deployment-setup#section-azure-setup) to complete the steps below.

2.  Launch a virtual machine in Azure. See [Creating Linux virtual machines](https://azure.microsoft.com/en-us/documentation/services/virtual-machines/linux/) for more details.

3.  Install Spinnaker on your virtual machine. For a Linux machine, execute the command:  **sudo bash -xc "$(curl -s https://raw.githubusercontent.com/spinnaker/spinnaker/master/InstallSpinnaker.sh)"**

4.  Specify azure as a cloud provider.  Choose a default region where you will deploy resources.

4.  Run the command **sudo apt-get update** then run **sudo apt-get upgrade spinnaker**

5.  Navigate to the directory by executing the command **cd /opt/spinnaker/config**

6.  Execute the command **ls** and you should see a file named **spinnaker-local.yml** in the config directory.

7.  We need to edit the spinnaker-local.yml file to add values for a [service principal](https://azure.microsoft.com/en-us/documentation/articles/active-directory-application-objects/). See [vim tutorial](https://linuxconfig.org/vim-tutorial) for more information on using the vim editor.

8.  Run the command **sudo vim spinnaker-local.yml**

9.  Type **a** to enter append mode in vim

10. Navigate to the Azure section and fill in details like the below. **Ensure you leave a space between the colon and the values you enter**.  These values were the ones you collected while creating a service principal for Azure:

    ${SPINNAKER_AZURE_ENABLED:**true**}

    defaultRegion: **&lt;insert region here. Example = westus&gt;**

    primaryCredentials:

    name: my-azure-account

    clientId: **&lt;ENTER YOUR CLIENT ID HERE&gt; Note:  You can run azure account show from the Azure CLI and obtain this from the username field**

    appKey: **&lt;ENTER YOUR APP KEY HERE  Note:  This was the password you created when creating the Azure service principal for Spinnaker**

    tenantId: **&lt;ENTER YOUR TENANT ID HERE&gt;**

    subscriptionId: **&lt;ENTER YOUR SUBSCRIPTION ID HERE&gt;**

11. Commit your changes with vim by clicking the **Esc** key, then type **:wq (colon wq)** and finally click enter.

12. Execute **sudo restart spinnaker** to restart the service.

13. Execute **curl http://localhost:9000** and verify you get a response.  Spinnaker is now installed and configured.

14. Execute **curl http://localhost:8084/health** and verify you get a response.

15. Configure SSH port forwarding on your local machine to test the setup.  For instructions for Linux see step 2 in the AWS docs here: [http://www.spinnaker.io/docs/creating-a-spinnaker-instance](http://www.spinnaker.io/docs/creating-a-spinnaker-instance).  For instructions on a Windows machine see below.

16. If you are using a Windows machine launch Putty and navigate to **Change Settings > SSH > Tunnels**.

17. In the Options controlling SSH port forwarding window, enter **8084** for Source port. Then enter **127.0.0.1:8084** for the Destination. Click **Add**. Repeat this process for ports: **8087** and **9000**, until you have all three listed in the text box for Forward ports. When finished, click **Open** to establish the connection.

18. On your local machine open a browser and navigate to **http://localhost:9000** where you should see a Spinnaker web page page render.
 
19. If you encounter issues see [Spinnaker Troubleshooting Guide]( http://www.spinnaker.io/docs/troubleshooting-guide "troubleshooting guide")

