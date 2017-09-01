---
layout: post
title: How to send Emails using PowerShell
---
 
<p>In this tutorial we will see how we can send emails using the tools provided by PowerShell</p>
<p>We are going to use Send-MailMessage utility provided by PowerShell to send emails in this tutorial.</p>
<p>We will also learn how to store the credentials securely in an XML file so that passwords are not stored directly in the source code</p>
<p>In this tutorial we will see how we can send emails using the tools provided by PowerShell</p>

We are going to use Send-MailMessage utility provided by PowerShell to send emails in this tutorial.

### Objective -
Send email to multiple recipients along with an attachment. Details of the emails we have to send are specified below:

| Parameter | Value |
| --- | --- |
| To | test_recipient@gmail.com|
| CC | test_cc@gmail.com|
| From | test_sender@gmail.com|
| Subject | Hello from Powershell |
|Body | Hello from Receiver|
| Attachment | A file named <em>mail.ps1</em> |

### Solution -
The usage of powershell Send-MailMessage command to send emails is as shown below. We are sending the email using Gmail SMTP server, but the details of any other SMTP server could be specified in the similar manner.

<script src="https://gist.github.com/amritanshu-pandey/da3904fc7274b9e4797801e7a19be225.js"></script>

In this example we hard-coded our gmail password in the script directly. This is not ideal as the password will be visible to the anybody who has the access to the source code. To solve this problem, now we will see how we can store our password in a file in an encrypted manner, which we can then use in our script.

We are now going to use following commands to take the password as input from the user and store that securely in an XML file.

| Command | Description |
| --- | ---| 
| Get-Credentials | This command take password as input from the user and create a credential object from that. |
| Export-Clixml | Command to create an XML based representation of an object and store that in a file |
| Import-Clixml | This command imports the CLIXML file created using Export-Clixml and create object in PowerShell |

We will combine the commands described above to take the password from user and store that in an XML file. Later on we will use the Import-Clixml command to read our password from the file.

Following code will store the credentials in `credentials.xml` file.
Note that the Get-Credentials command store the password as a secured system string which is encrypted using the Windows user information and hence is intelligible only to the windows user which was used to input the password.

```
Get-Credential | Export-Clixml credentials.xml
```
Enter the username and password when prompted for.
```
cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
User: test_sender@gmail.com
Password for user test_sender@gmail.com: *************
```
Now we will modify the Send-MailMessage command used before to take the password from the credentials.xml file created before.

<script src="https://gist.github.com/amritanshu-pandey/b064b534c6e7e39f93a26236076b7d5e.js"></script>

Send-MailMessage cmdlet provide several other useful functionalities related to sending emails, like sending HTML emails, keeping blind carbon copies in the mails etc. I would suggest you use `Get-Help` command in PowerShell to know more about the commands used in this tutorial.

```
Get-Help Send-MailMessage
```
