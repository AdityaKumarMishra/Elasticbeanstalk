# Deploying a Application to AWS Elastic Beanstalk


###### Elastic Beanstalk is one of the plethora of services provided by Amazon Web services (AWS) which provides you with the capability to run scalable and highly available applications in no time. It’s a one-stop solution for you if you want a self-managed cloud infrastructure on AWS.	

Deploying manually on Beanstalk using a ZIP file

CREATING YOUR BEANSTALK ENVIRONMENT

  1. Sign in into your AWS Account in any Region and navigate to AWS Elastic Beanstalk in the services section. Please make sure you have the Beanstalk Full Access IAM policy attached with your account.

  2. Click on Create New Application and give a possible description of your application.

  3. Click on Create one now to create an environment and select Web Server Environment.

  4. Type in a domain name of your choice and check if it’s available. This domain will be different from the one for which we will generate HTTPS certificates using certbot.
 
  5. Choose Python as the language in the list of preconfigured Platforms.

  6. Choose Upload your code as an option to deploy your code.
 
  7. Head over to the Directory in which you have the code on your system.
 
  8. If not, the status of the app may be Info, Severe or Degraded. In either of the cases, you can click on the Causes button and see the possible list of errors. Or you can SSH into your application servers using the pem file you selected and try to see the log files(either of access_log, error_log(in /var/log/httpd)or eb-activity.log(/var/log/eb-activity.log).Happy Debugging!


**When you create an environment, AWS Elastic Beanstalk prompts you to provide two AWS Identity and Access Management (IAM) roles: a service role and an instance profile. The service role is assumed by Elastic Beanstalk to use other AWS services on your behalf. The instance profile is applied to the instances in your environment and allows them to retrieve application versions from Amazon Simple Storage Service (Amazon S3), upload logs to Amazon S3, and perform other tasks that vary depending on the environment type and platform. Create an environment running a sample application in the Elastic Beanstalk console or by using the Elastic Beanstalk Command Line Interface (EB CLI). When you create an environment, the clients create the required roles and assign them managed policies that include all of the necessary permissions.**

`In addition to the two roles that you assign to your environment, you can also create user policies and apply them to IAM users and groups in your account to allow users to create and`

manage Elastic Beanstalk applications and environments. Elastic Beanstalk provides managed policies for full access and read-only access.
You can create your own instance profiles and user policies for advanced scenarios. If your instances need to access services that are not included in the default policies, you can add additional policies to the default or create a new one. You can also create more restrictive user policies if the managed policy is too permissive. See the AWS Identity and Access Management User Guide for in-depth coverage of AWS permissions.

*Set up an IAM User*

The next thing we need to do is set up an IAM user. This will give us access to API keys we can use to SSH and access our resources from outside AWS.
It is also a good idea to have a separate IAM user with access to only the resources the user needs instead of using the default admin credentials for security purposes.
On the home page, search for IAM and navigate to the *IAM home page*.



On the *IAM homepage*, under *AM Resources*,click on *Users: 0*
.
After that, click *Add User*.You can fill out the user name of your choice and then select the checkbox for *Programmatic access*.
On the next screen, select *Attach existing policies directly* and then use the search box to search for *Administrator Access*.
On the next page, add a name tag so you can identify your user later from the list of IAM credentials:

Finally, on the review page, click on*Create User*.

On the next page, download the CSV file using your credentials. We will need them for the last part.

Once you have the file called *credentials.csv*,you can open it in any
 spreadsheet app or editor to see the values in it. We are mostly interested in *Access key ID* and *Secret access key*.

The last thing you need to do is to go to your HOME folder and create a folder called .aws. Inside this folder, place a file called con fig. Notice that the folder name starts with a . and the file has no extension. The full path should be something like /Users/your-user/.AWS/con fig.

If you are unable to create the .aws folder and configure file, you can skip it for now. The important thing is to have the CSV file with your credentials on hand for later use.

Place the following into the con fig file:

[profile eb-cli]
*region = us-east-1*
*aws_access_key_id = your-AWS-access-key-id*
*aws_secret_access_key = your-AWS-secret-access-key*

You can find your AWS region on the top-right corner of the AWS account page when you sign in


###### Create Elastic beanstalk App

Navigate to the elastic beanstalk home page and click Create New Application.


Fill out the new application form as necessary.

You will then see a page with the message *No environments currently exist for this application*. Create one now. Click on *Create one now*.
Next, select *Web server environment*.



In the next section, change the Environment name to production-env. Leave the Domain blank. Then, under Base Configuration, select Preconfigured platform and choose Ruby from the dropdown. You can leave Application code on Sample application; then, go ahead and click Create environment. This will take some time, so be patient.












Once done, you should see a page that looks like this:
Find the URL provided after the environment ID. Click on it to check out the default Ruby app on elastic beanstalk. Very soon, your API will be running on the same URL.
You can change the version label to something more meaningful:

Once Elasticbeanstalk has finished updating the environment, you can visit your environment URL to see your API running! 

**Deployment with EB CLI**

The Elastic beanstalk CLI makes most of what I have had to do manually up to this point quite easy to accomplish with a few commands. How to set it up and use it on our current project. This all depends on having your IAM user set up correctly, so make sure everything from that step is okay or that you have the CSV with your credentials ready.

For most computers, you should already have Python installed. Installation will, therefore, be as easy as the following:

Install Python, `pip`, and the `EB CLI` on Linux

The EB CLI requires `Python 2.7, 3.4`, or `later`. If your distribution didn't come with *Python*, or came with an earlier version, install *Python* before installing*pip*and the *EB CLI*.
Determine whether Python is already installed.
``$ python --version``

Note
If your Linux distribution came with Python, you might need to install the Python developer package to get the headers and libraries required to compile extensions and install the EB CLI. Use your package manager to install the developer package (typically named python-dev or python-devel).
If Python 2.7 or later isn't installed, install Python 3.7 using your distribution's package manager. The command and package name vary:

    • On Debian derivatives, such as Ubuntu, use APT.
      $ sudo apt-get install python3.7
    • 
    • On Red Hat and derivatives, use yum.
      $ sudo yum install python37

To verify that Python installed correctly, open a terminal or shell and run the following command.

       `$ python3 --version`
       
       Python 3.7.3

Install`pip`by using the script provided by the Python Packaging Authority, and then install the EB CLI.
To install`pip`and the `EB CLI`

    1. Download the installation script from pypa.io.
       $ curl -O https://bootstrap.pypa.io/get-pip.py
       
    2. The script downloads and installs the latest version of pip and another required package named setuptools.
    3. 
    4. 
    5. Run the script with Python.
       $ python3 get-pip.py --user
       Collecting pip
         Downloading pip-8.1.2-py2.py3-none-any.whl (1.2MB)
       Collecting setuptools
         Downloading setuptools-26.1.1-py2.py3-none-any.whl (464kB)
       Collecting wheel
         Downloading wheel-0.29.0-py2.py3-none-any.whl (66kB)
       Installing collected packages: pip, setuptools, wheel
       Successfully installed pip setuptools wheel


    6. Invoking Python version 3 directly by using the python3 command instead of python ensures that pip is installed in the proper location, even if an earlier version of Python is present on your system.
    7. 
    8. Add the executable path, ~/.local/bin, to your PATH variable.

       To modify your PATH variable (Linux, Unix, or macOS):
    9. 
        a. Find your shell's profile script in your user folder. If you are not sure which shell you have, run echo $SHELL.
           $ ls -a ~
           .  ..  .bash_logout  .bash_profile  .bashrc  Desktop  Documents  Downloads
            ▪ 
            ▪ Bash – .bash_profile, .profile, or .bash_login.
            ▪ 
            ▪ Zsh – .zshrc
            ▪ 
            ▪ Tcsh – .tcshrc, .cshrc or .login.
            ▪ 
        b. Add an export command to your profile script. The following example adds the path represented by LOCAL_PATH to the current PATH variable.
        c. 
           export PATH=LOCAL_PATH:$PATH
        d. 
        e. Load the profile script described in the first step into your current session. The following example loads the profile script represented by PROFILE_SCRIPT.
        f. 
           $ source ~/PROFILE_SCRIPT
    10. 
    11. 
    12. Verify that pip is installed correctly.
       $ pip –version
    13. 
       pip 8.1.2 from ~/.local/lib/python3.7/site-packages (python 3.7)

    14. 
    15. Use pip to install the EB CLI.
       $ pip install awsebcli --upgrade --user


    16. Verify that the EB CLI installed correctly.
       $ eb --version
       EB CLI 3.14.8 (Python 3.7)


To upgrade to the latest version, run the installation command again.
``$ pip install awsebcli --upgrade --user``


Once the installation is finished, cd to your project folder and run eb init.
Follow the prompts as per your AWS environment. Select a region by entering the correct number selection:

*Select a default region*

1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-south-1 : Asia Pacific (Mumbai)
7) ap-southeast-1 : Asia Pacific (Singapore)
8) ap-southeast-2 : Asia Pacific (Sydney)
9) ap-northeast-1 : Asia Pacific (Tokyo)
10) ap-northeast-2 : Asia Pacific (Seoul)
11) sa-east-1 : South America (Sao Paulo)
12) cn-north-1 : China (Beijing)
13) cn-northwest-1 : China (Ningxia)
14) us-east-2 : US East (Ohio)
15) ca-central-1 : Canada (Central)
16) eu-west-2 : EU (London)
17) eu-west-3 : EU (Paris)

*Pick any one*

Go through the rest of the prompts and select 'no' to using CodeCommit and 'no' to set up SSH.

If you did not set up the AWS config, the CLI will prompt you for your AWS keys

Add them as required.

Once this is done, the CLI will exit. You can then run commands, such as `eb status`,to check the status of the app we deployed.

Environment details for: production-env

  `Application name: movie-api`
   `Region: us-east-2`
 `` Deployed Version: Deploy 2-2``
 `` Environment ID: e-mab3kjy6pp``
   `Platform: arn:aws:elasticbeanstalk:us-east-2::platform/Puma with Ruby 2.6`
   `unning on 64bit Amazon Linux/2.11.1`
   `Tier: WebServer-Standard-1.0
   `CNAME: production-env.qnbznvpp2t.us-east-2.elasticbeanstalk.com`
   `Updated: 2020-1-22 23:37:17.183000+00:00
   `` Status: Ready``
    `Health: Green

To deploy a new version, simply run `eb deploy.`
