---
title: Social Sign On with Google Enterprise in Salesforce Org
layout: blog
category: [Salesforce]
excerpt: In this blog, we will explain the steps of enabling social sign on into salesforce enterprise org using your corporate Gmail account. Scenario - John is an employee of an enterprise called “XYZ.com”. The company uses Salesforce as their primary CRM. Just like John, there are many other employees who face issues with remembering username and...
comments: true
---

In this blog, we will explain the steps of enabling social sign on into salesforce enterprise org using your corporate Gmail account.

## Scenario:

John is an employee of an enterprise called “XYZ.com”. The company uses Salesforce as their primary CRM. Just like John, there are many other employees who face issues with remembering username and password for logging into their Salesforce CRM. As per company policy, no employees are supposed to use any Password storing apps/even write down this sensitive information on their notebooks/whiteboards.

## The solution proposed by the company:

The company proposes to enable Social Sign-On capability with which every employee will be able to use their corporate Gmail login credentials to login to Salesforce CRM. With this technique, a new employee provisioned in Gmail is automatically allowed into Salesforce CRM. Any employee who leaves the organization is also denied the Salesforce CRM login.

## Steps to implement Social Sign-On in Salesforce CRM

For configuring this feature, we will be using the Open ID Connect protocol in conjunction with Authentication Providers in Salesforce. With this setup, Salesforce behaves like an Open ID Client.

1. Open [Google Console](https://console.developers.google.com)
2. Create an App and a Credential of type “Web application”
3. This will generate a Client ID & Client Secret. Keep this handy as we will use this data in the upcoming step.
4. Login to Salesforce CRM [assuming that you are the super admin]
5. Go to Setup
6. Quick Find “My Domain” and open it
7. Set up a domain name, if not yet done. It might take a few minutes for the changes to be active. Be patient.
8. Once the domain is set up, the MyDomain screen will look like:
9. Now Quick Find “Auth” and open “Auth. Providers” option.
10. Click “New”
11. Select Provider Type as “Open ID Connect”
12. Fill up the other details as shown in the screenshot below. Use the Client ID and Client Secret that you have from Step no.3.
13. For Registration Handler, give the name as “GoogleEnterpriseSignOn” and allow this wizard to create the class by itself.
14. Save the Auth Provider. Now, you will see the “Salesforce Configuration” section in this Auth Provider populate with many URL’s.
15. Copy the “Callback URL” from the Auth Provider and paste it in the “Authorized redirect URIs” box of the Google app that you created in Step no. 3. Finally hit the “Save” button.
16. Now, open up the “GoogleEnterpriseSignOn” apex class and paste the following code here. The default code by default won’t work.
17. Feel free to change the code in line no.28 that checks for a username ending with “xyz.com”. These are some enterprise code checks that you can do to ensure that only your company employees are able to access Salesforce CRM and not anyone who has a Gmail account.
    ```apex
    global boolean canCreateUser(Auth.UserData data)
    {
        if(data.email!=null)
        {
            return true;
        }
        else
        {
            return false;
        }
    }
    global User createUser(Id portalId, Auth.UserData data)
    {
        if(canCreateUser(data))
        {
            List users = [select Id from User where Google_ID__c=:data.identifier];
            if(users.size()==1)
            {
                debug('#1##'+users[0]);
                return users[0];
            }
            else
            {
                User u = new User();
                Profile p = [SELECT Id FROM profile WHERE name='System Administrator'];
                username = data.email.substring(0,data.email.indexOf('@'))+ '@xyz.com';
                email = data.email;
                lastName = data.lastName;
                firstName = data.firstName;
                String alias = data.firstName.substring(0,1)+data.lastName.substring(0,4);
                if(alias.length() > 8)
                {
                    alias = alias.substring(0, 8);
                }
                alias = alias;
                languagelocalekey = 'en_US';
                localesidkey = 'en_US';
                emailEncodingKey = 'UTF-8';
                timeZoneSidKey = 'America/Los_Angeles';
                profileId = p.Id;
                Google_ID__c = data.identifier;
                insert u;
                return u;
            }
        }
        else
        {
            return null;
        }
    }
    global void updateUser(Id userId, Id portalId, Auth.UserData data)
    {
        User u = new User(Id=userId);
        //u.username = data.email.substring(0,data.email.indexOf('@'))+ '@salesforce.com';
        //u.Google_ID__c = data.identifier;
        update u;
    }
    ```
18. Save the apex class.
19. Now, open a new incognito browser session and paste the Test-Only Initialization URL that you obtained from the “Auth Provider” in step no. 13
20. If you have done all these steps correctly, then running the Test-Only Initialization URL in a new incognito browser session should redirect you to the Google Login page. If you provide your corporate Gmail username and password, it should authorize and show you an XML as shown below.
21. This testing confirms that you have done all the steps correctly.
22. Now, switch back to Salesforce –> Setup –> Quick Find “My Domain” and open it
23. Under the “Authentication Configuration” section, click “Edit” button
24. Tick mark the “Google Enterprise” option. This is nothing but your newly created Auth Provider.
25. Save the screen.
26. Logout from Salesforce
27. On the main screen, now you will be able to see a “Google Enterprise” option for employees to click. Using this option, employees can log in to Salesforce instead of typing the salesforce username/password.
28. To understand how we can write a test class for the AuthProvider, let’s see the test class code written below.
    ```apex
    @isTest
    private class GoogleEnterpriseSignOnTest
    {
        static testMethod void testCreateAndUpdateUser()
        {
            GoogleEnterpriseSignOn handler = new GoogleEnterpriseSignOn();
            Auth.UserData sampleData = new Auth.UserData('testId', 'testFirst', 'testLast',
            'testFirst testLast', 'testuser@example.org',
            null, 'testuserlong', 'en_US', 'google',
            null, new Map<String, String>{'language' => 'en_US'});
            User u = handler.createuser(null, sampleData);
            System.assertEquals('testuser@xyz.com', u.userName);
            System.assertEquals('testuser@example.org', u.email);
            System.assertEquals('testLast', u.lastName);
            System.assertEquals('testFirst', u.firstName);
            System.assertEquals('ttest', u.alias);
            System.assertEquals('testId', u.Google_ID__c);
            String uid = u.id;
            sampleData = new Auth.UserData('testNewId', 'testNewFirst', 'testNewLast',
            'testNewFirst testNewLast', 'testnewuser@example.org', null, 'testnewuserlong', 'en_US', 'facebook',
            null, new Map<String, String>{});
            handler.updateUser(uid, null, sampleData);
            //User updatedUser = [SELECT userName, email, firstName, lastName, alias FROM user WHERE id=:uid];
            //System.assertEquals('testnewuserlong@salesforce.com', updatedUser.userName);
            //System.assertEquals('testnewuser@example.org', updatedUser.email);
            //System.assertEquals('testNewLast', updatedUser.lastName);
            //System.assertEquals('testNewFirst', updatedUser.firstName);
            //System.assertEquals('testnewu', updatedUser.alias);
        }
    }
    ```

Writing a test class is very simple as you just have to set the necessary properties of the Auth.UserData and feed that into the createUser & updateUser method of your main AuthProvider class. I have commented a few lines of the updateUser asserts because I do not have a logic written inside the updateUser method of GoogleEnterpriseSignOn AuthProvider class. With this simple test class, I was able to achieve 87% code coverage. This test class template would help you to get started. You can refine & tune the test class as per your custom logic. Hope this helps. Click here to learn more about Registration Handlers in Salesforce. I hope this blog was informative. Try it out folks and let me know if you have any questions/concerns.
