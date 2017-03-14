First, you'll need to have an AWS project set up. If you've already got one, please skip to the next step. Otherwise, please follow the instructions below.

Keep in mind that naming of your entities in AWS is important as Spinnaker will use them to populate available resource lists in the Spinnaker UI.

Sign into the [AWS console](https://console.aws.amazon.com) and let AWS pick a default region where your project resources will be allocated. In the rest of this tutorial, we'll assume that the region
assigned is <code>us-west-2</code>. If the region selected for your project is different from this, please substitute your region everywhere <code>us-west-2</code> appears below.

Also, in the instructions below, we'll assume that your AWS account name is <code>my-aws-account</code>. Wherever you see <code>my-aws-account</code> appear below, please replace it with your AWS account name.

1. Create VPC.
  * Goto [Console](https://console.aws.amazon.com) > VPC.
  * Click on **Start VPC Wizard**.
  * On the **Step 1: Select a VPC Configuration** screen, make sure that **VPC with a Single Public Subnet** is highlighted and click **Select**.
  * Name your VPC. Enter <code>defaultvpc</code> in the **VPC name** field.
  * Enter <code>defaultvpc.internal.us-west-2</code> for **Subnet name**.
  * Click **Create VPC**.

2. Create an EC2 role.
  * Goto [Console](https://console.aws.amazon.com) > AWS Identity & Access Management > Roles.
  * Click **Create New Role**.
  * Set **Role Name** to <code>BaseIAMRole</code>. Click **Next Step**.
  * On **Select Role Type** screen, hit **Select** for **Amazon EC2**.
  * Click **Next Step**.
  * On **Review** screen, click **Create Role**.
  * EC2 instances launched with Spinnaker will be associated with this role.

3. Create an EC2 Key Pair for connecting to your instances.
  * Goto [Console](https://console.aws.amazon.com) > EC2 > Key Pairs.
  * Click **Create Key Pair**.
  * Name the key pair <code>my-aws-account-keypair</code>. (Note: this must match your account name plus "-keypair")
  * AWS will download file <code>my-aws-account-keypair.pem</code> to your computer. <code>chmod 400</code> the file.

4. Create AWS credentials for Spinnaker.
  * Goto [Console](https://console.aws.amazon.com) > AWS Identity & Access Management > Users > Create New Users. Enter a username and hit **Create**.
  * Create an access key for the user. Click **Download Credentials**,
    then Save the access key and secret key into
    <code>~/.aws/credentials</code> on your machine as shown
    [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-config-files).
  * Click **Close**.
  * Click on the username you entered for a more detailed screen.
  * On the **Summary** page, click on the **Permissions** tab.
  * Click **Attach Policy**.
  * Click the checkbox next to **PowerUserAccess**, then click **Attach Policy**.
  * Click on the **Inline Policies** header, then click the link to create an inline policy.
  * Click **Select** for **Policy Generator**.
  * Select **AWS Identity and Access Management** from the **AWS Service** pulldown.
  * Select **PassRole** for **Actions**.
  * Type <code>*</code> (the asterisk character) in the **Amazon Resource Name (ARN)** box.
  * Click **Add Statement**, then **Next Step**.
  * Click **Apply Policy**.

