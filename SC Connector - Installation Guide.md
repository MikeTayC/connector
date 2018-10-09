# SC Connector: Installation Guide


## Service Cloud Connector: Overview

The Service Cloud (SC) Connector is a reusable code asset to support the enablement and acceleration of specific integration use cases between Commerce Cloud and Service Cloud. SiteGenesis is used as a reference storefront for CC Connector cartridge integration.

To run Service Cloud Connector,  we need at least one Commerce Cloud instance and one Service Cloud org. Installation is basically a 3-step process:

**Step 1:** Install SCC Salesforce package on your Service Cloud org. 

**Step 2:** Import all provided SFCC configuration metadata on your Commerce Cloud instance. 

**Step 3**: Upload commerce cloud cartridges and add them in respective site class path. 

Please, see below for detailed description of each installation step.


## **Step 1. Service Cloud: Set up and Configuration**

### **1.  User set up prior to installation of  SCC Connector Salesforce package**

**1.1.A Create SCC User Profile**. Prior to installing SCC Connector Salesforce package, create the following required custom profile: “SCC Integration User”.  See screenshot below for profile configuration specifics.

Select “System Administrator” as the existing profile to clone from under Setup→Profiles→New Profile. Click “Save”.
[[images/installation-setup-1.png]]

[[images/installation-setup-2.png]]


On the “SCC Integration User” profile page, make sure that “API Enabled” checkbox is checked, as shown below.

[[images/installation-setup-3.png]]

**1.1.B Create SCCIntegration user.**
Now that you created respective user profile (see (1.A), create SCCIntegration user  (Setup→Users) on Service Cloud to connect from Commerce Cloud. SCCIntegration user will need to have “User Licence” to 'Salesforce' and to use “SCC Integration User” custom profile. 


See below for details. 
[[images/installation-setup-4.png]]

**1.1.C Create “CommerceCloudConnector” Connected app.**
Login using newly created SCCIntegration user. (Make sure you logged in using integration user)

In SF Classic: Go to Setup → Create → APP. Click new from button in Connected app section at bottom on page.
In SF Lightning: Go to SETUP → App Manager → New Connected App
Create a new connected app and name it “CommerceCloudConnector”
Api name will be automatically populated. 
Use your own admin email id (not SCCIntegration user email id). 
Select “**Enable OAuth Settings**” and “**Enable for Device Flow**”.
From “**Selected OAuth Scopes**” section select “Full access” and save it
[[images/installation-setup-5.png]]

Once you save connected app. it will show following message 

[[images/installation-setup-6.png]]
**NOTE: **Click on Continue button and it will show you below page with Consumer key (Client id) and Consumer secret (Client secret). You can reveal it using hyperlink. Note down both values as we will needing these values to configure connection in Commerce Cloud Business manager configuration. 

[[images/installation-setup-7.png]]
Please note that Connected app can take up to 10-15 minutes sometimes to start working. Usually it gets setup immediately. So give it time if integration fails in commerce cloud and you cannot get auth token as described in commerce cloud installation steps. 

Now you have created profile, user and connected app for integration with commerce cloud.  One last step here is to update IP restrictions for CommerceCloudConnector app to “Relax IP Restriction” setting under  “Manage App”.

[[images/installation-setup-8.png]]


**NOTE: **Once integration user and connected app is created,  logout SCCIntegration user from Salesforce and use your own Salesforce user to login in Salesforce. This integration user is to be used for integration purposes only, and not to be used by any SF business user to operate on SF. Make sure you are NOT logged in as the integration user for the subsequent configuration steps below.

### **1.2 Install SCC Salesforce package on your Service Cloud org. **


** Use this URL to install the SCConnector package for Service Cloud into any organization:
https://login.salesforce.com/packaging/installPackage.apexp?p0=04t1I000003NNtT **

**IMPORTANT: When prompted, select "Install for All Users" radio button.**

On “Approve Third-Party Access” screen, select “Yes, grant access to these third-party web sites”. 
[[images/installation-setup-9.png]]

Note, that the Website has the following test sandbox: nyadav-inside-eu02-dw.demandware.net. In current release it serves as a placeholder. Please see below for how to update the text sandbox URL to your SFCC instance in Remote Site Configuration in Service Cloud.

If installed successfully, you will be able to see in App Launcher the following two apps (see below):

* SCC Call Center Lightning (console app, for call center users)
* SCC Service Cloud Lightning (for business users)
* SCC Call Center (deprecated console app, this is for Salesforce Classic)

[[images/installation-setup-10.png]]

** 1.2.1 Give access to Connected App under profile: “SCC Integration User”**
Make sure you are logged in as the Integration user. See below.

[[images/installation-setup-11.png]]

**NOTE: **Once integration user and connected app is created,  logout SCCIntegration user from Salesforce and use your own Salesforce user to login in Salesforce. This integration user is to be used for integration purposes only, and not to be used by any SF business user to operate on SF. Make sure you are NOT logged in as the integration user for the subsequent configuration steps below.

### **1.3 Remote Site Configuration in Service Cloud**

After installing SCC package, go to Setup → Remote Site Settings. Update  “Remote Site URL” to your SFCC instance for the existing SCC_End_Point remote site for installed package 'SCCConnector'. See below.

[[images/installation-setup-12.png]]

### **1.4 Verify 'CommerceCloud' Connected App.**

Verify that you are observing 'CommerceCloud' in the list of Connected Apps on your Service Cloud instance. Go to Setup→ Apps→ AppManager and check for 'CommerceCloud'. See below.

[[images/installation-setup-13.png]]



### **1.5 Authentication Settings**

**1.5.1 Create Named Credentials**

Navigate to Setup->Named Credentials. Observe that “SFCCClientCreds” credential was created with the package install. Next, click “New Named Credential” and create “SFCCUserCreds” credential (using same name and case!):

* For **URL**, use domain name of your SFCC instance.
* Set “Identity Type” to “Per User”. 

See below for example:
[[images/installation-setup-14.png]]

Go back to Named Credentials. Next, click “New Named Credential” and create “SFCCClientCredsBearer” credential (using same name and case!):

* For *URL*, use domain name of your SFCC instance.
* Set “Identity Type” to “Named Principal”
* Set “Authentication Protocol” to “No Authentication” 
* Uncheck “Generate Authorization Header”
* Check “Allow Merge Fields in HTTP Header” and “Allow Merge Fields in HTTP Body”

See below for example:

[[images/installation-setup-15.png]]

**1.5.2 Authentication setting for external systems**

Add your Commerce Cloud user credentials in  “Authentication Settings for External Systems” in Service Cloud.  After logging into Service Cloud, navigate to “Settings”, as shown below.
[[images/installation-setup-16.png]]

On “Settings” page select “Authentication Settings for External systems” from left menu. 

Create new external system settings, as follows.

* Select SFCCUserCreds as Named Credential. 
* Select your own user in service cloud in user field. 
* Enter your Commerce Cloud instance user name and password in respective fields. **IMPORTANT: **Value in password field has the following format: **{your commerce cloud password}:{OCAPI Client id}.** OCAPI Client ID needs to correlate with the client ID configured for your SFCC instance. See below.

[[images/installation-setup-17.png]]

*IMPORTANT:* *Repeat this step for the integration user and for all the agent's account that will need to have access to Service Cloud.*

### **1.6 Update SFCC Site info in SCC Connector Custom Settings**


Navigate to Setup→ Custom Settings, select “Manage” next to “SFCC Configuration” (see below).

[[images/installation-setup-18.png]]

Click “New”. See below for sample configuration:
[[images/installation-setup-19.png]]

**NOTE**: Empty fields will be used in future releases. “SFCC Site URL” is only your site's domain name (e.g. [https://oparkhomenko-inside-na02-dw.demandware.net](https://oparkhomenko-inside-na02-dw.demandware.net/)). “Time” is a scheduling time interval to pull data from SFCC, once draft data is created by SFCC into SF. Value of 1 here refers to 1 minute.

Next, update “SFCC Integration Creds” configuration. Click “Manage” and click on the “New” button located above the "Default Organization Level Value" section. Values will be pre-populated from SiteGenesis as follows:

[[images/installation-setup-20.png]]

Update Client ID and Client Secret to merchant's correct values or leave as is if testing for SiteGenesis. Make sure to Save this configuration.

### **1.7  Configure Page Layout for Contact and Order page views**

**1.7.1 Contact Page Layout Assignment**
Make sure you are logged in as dev user on Service Cloud (not Integration user) and navigate to Setup→ObjectManager→Contact→Page Layouts→ Page Layout Assignment. Click on “Edit Layout” button. 
Then locate profile of your logged in user  (e.g. System Administrator) and select it. In the “Page Layout to use” drop down, select Contact (SCC - Support) Layout.

See below.
[[images/installation-setup-21.png]]

**1.7.2 Order Page Layout Assignment**
Now, repeat same steps for Order object and set “Order (SCC Support) Layout”. See below.
[[images/installation-setup-22.png]]

Then select 'Compact Layouts'  → Compact Layout Assignment → Edit Assignment, select 'SCC Order Compact Layout' and click on Save.

[[images/installation-setup-23.png]]

Go to '*Fields & Relationship*', select 'Status' and add the steps for “*Order Status Picklist Values*” section (see screenshot below):

* Label: 'New' 
* API Name: 'New'
* Status Category: 'Draft'

and

* Label: 'Open' 
* API Name: 'Open'
* Status Category: 'Draft'

[[images/installation-setup-24.png]]

**1.8  Configure Login on Behalf (OOBO module)**

In order to grant the rights to allow logging in the storefront on behalf of the customer you need to add the following configuration.

**1.8.1 CORS**

Go to  Setup → Security →  CORS  and add a new Whitelisted Origin with the URL of the SFCC instance you are working with.

[[images/installation-setup-25.png]]

**1.8.2 Content Security Policy Trusted site**

Go to Setup → Security → CSP Trusted Sites and click on “New Trusted Site” , then add the following values

* Trusted Site Name: CC_Sandbox
* Trusted Site URL : your SFCC instance url

[[images/installation-setup-26.png]]

**1.8.3 Domain**
Go to Setup→Company Settings → My Domain and register a domain for your Salesforce URL

[[images/installation-setup-27.png]]

## **Step 2 & 3. Commerce Cloud: Set up and Configuration**

### **Step 2: Import all provided SFCC configuration metadata on your Commerce Cloud instance. **

**2.A **In Business Manager, go to Administration > Site Development > Import & Export. Upload and import two files:

* [system-objecttype-extensions.xml](https://bitbucket.org/demandware/service-cloud-connector/src/fcb1edc69da606538158ee412a7abb9bcb028ce6/sites/site_template/meta/system-objecttype-extensions.xml?at=master)
* [custom-objecttype-definitions.xml](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b930bdea7098b040eb711ebfcb67985/sites/site_template/meta/custom-objecttype-definitions.xml?at=master)


Under “Custom Site Preferences”, add your SCC Integration User (set up for Service Cloud instance) user name and password, as shown below:
[[images/installation-setup-28.png]]

NOTE: Make sure  that **async **mode is set to a value (Yes/No) and not left as “None”, as general functionality will be disabled.

**2.B **In Business Manager, go to Administration > Operations> Import & Export. Upload and import [services.xml](https://bitbucket.org/demandware/service-cloud-connector/src/fcb1edc69da606538158ee412a7abb9bcb028ce6/sites/site_template/services.xml?at=master). 
**Service credential configuration:**
Note that you will receive the following warning after import completes: “ServiceCredential: servicecloud.rest.auth: Service credentials contain a masked password, skipping this field”.  You will need to enter those credentials manually. 
Password is your Consumer secret (Client Secret) noted from “CommerceCloudConnector” Connected App setup in Step 1.C above and username is Consumer key (Client id).  You can view these credentials in SF Lightning unde Setup->Manage Connected Apps→ CommerceCloudConnector app→ API (Enable OAuth Settings) section.

In Business Manager of your SFCC instance, navigate to Administration > Operations > Services > **servicecloud.rest.auth **and enter User ID (Consumer Key) and Password (Consumer secret). See below.

[[images/installation-setup-29.png]]
**NOTE:  **In the above service credential name, the “-SiteGenesis” is the site ID. You will need to update to your storefront/site ID in Service credential name.

**2.C Verify OCAPI JSON Configuration on SFCC instance**

Add Salesforce domain name of your working instance to the “allowed_origins” for "Shop API" of your site context. See below for example:

[[images/installation-setup-30.png]]
Make sure the following API config is present on your SFCC instance. See below.

**Data API (at the Global, org-wide context)**

```
{
  "_v":"17.8",
  "clients":
  [ 
    {
      "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "resources":
      [
         {
          "resource_id":"/customer_lists/*",
          "methods":["get"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        }, 
         {
          "resource_id":"/customer_lists/*/customers/*",
          "methods":["get","put","patch","delete"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        }, 
        {
          "resource_id":"/customer_lists/*/customer_search",
          "methods":["post"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        }, 
        {
          "resource_id":"/customers",
          "methods":["post"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        },
        {
          "resource_id":"/customers/*",
          "methods":["get","patch","delete"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        },
        {
          "resource_id":"/customers/*/addresses",
          "methods":["get","post"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        },
        {
          "resource_id":"/customers/*/addresses/*",
          "methods":["get","patch","delete"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        },
        {
          "resource_id":"/customer_search",
          "methods":["post"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        }
      ]
    }
  ]
}
```


**Shop API (at the Site context)**

```
{
  "_v":"16.1",
  "clients":
  [
    {
      "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
      "allowed_origins":["https://nyadav-inside-eu02-dw.demandware.net", "https://eu6.salesforce.com", "https://neeeraj.my.salesforce.com"],
      "resources":
      [
        {
          "resource_id":"/orders/*",
          "methods":["get","patch"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        },
        {
          "resource_id":"/customers/auth",
          "methods":["post"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        },
        {
          "resource_id":"/customers/*/auth",
          "methods":["post"],
          "read_attributes":"(**)",
          "write_attributes":"(**)"
        }
      ]
    }
  ]
}

```


### **Step 3: Upload commerce cloud cartridges and add them in respective site path. **

**Step 3.1  Install SC Connector cartridge**
Bitbucket repository: https://bitbucket.org/demandware/service-cloud-connector
SC Connector integration cartridge: **int_service_cloud**
Upload the cartridge to your active code deployment version and configure “Effective cartridge path” in include int_service_cloud** **under Administration > Sites > Manage Sites > [Your site name] - Settings.

**Step 3.2 Modifications to storefront cartridge(s)**
The following three script files in your storefront cartridge(s) need to have modifications listed below. 

**(1) CustomerService.js: Add below snippet to submit() function**

```
        /**
         * @type {dw.system.HookMgr}
         */
        const HookMgr = require('dw/system/HookMgr');
        var hookID = 'app.case.created';
        if (HookMgr.hasHook(hookID)) {
            HookMgr.callHook(
                hookID,
                hookID.slice(hookID.lastIndexOf('.') + 1),
                contactUsForm
            );
        } else {
            require('dw/system/Logger').debug('no hook registered for {0}', hookID);
        }


```

Here's complete submit() function with the modification:


```
function submit() {
    var contactUsForm = app.getForm('contactus');

    var contactUsResult = contactUsForm.handleAction({
        send: function (formgroup) {
            // Change the MailTo in order to send to the store's customer service email address. It defaults to the
            // user's email.
            var Email = app.getModel('Email');
            return Email.get('mail/contactus', formgroup.email.value)
                .setFrom(formgroup.email.value)
                .setSubject(formgroup.myquestion.value)
                .send({});
        },
        error: function () {
            // No special error handling if the form is invalid.
            return null;
        }
    });

    if (contactUsResult && (contactUsResult.getStatus() === Status.OK)) {
           /**
         * @type {dw.system.HookMgr}
         */
        const HookMgr = require('dw/system/HookMgr');
        var hookID = 'app.case.created';
        if (HookMgr.hasHook(hookID)) {
            HookMgr.callHook(
                hookID,
                hookID.slice(hookID.lastIndexOf('.') + 1),
                contactUsForm
            );
        } else {
            require('dw/system/Logger').debug('no hook registered for {0}', hookID);
        }
        
        app.getView('CustomerService', {
            ConfirmationMessage: 'edit'
        }).render('content/contactus');
    } else {
        app.getView('CustomerService').render('content/contactus');
    }
}
```



**(2) CustomerModel.js: Add below snippet to createAccount function**

```

    /**
     * @type {dw.system.HookMgr}
     */
    const HookMgr = require('dw/system/HookMgr');
    var hookID = 'app.account.created';
    if (HookMgr.hasHook(hookID)) {
        HookMgr.callHook(
            hookID,
            hookID.slice(hookID.lastIndexOf('.') + 1),
            newCustomer.object
        );
    } else {
        require('dw/system/Logger').debug('no hook registered for {0}', hookID);
    }
```

Here's complete CustomerModel.createAccount = function (email, password, form) with the modification:

```
/**
 * Creates a new customer account.
 * @transactional
 * @alias module:models/CustomerModel~CustomerModel/createAccount
 * @param {String} email - The current customer email address to use for login.
 * @param {String} password - The password of the customer.
 * @param {dw.web.Form} form - The form instance to invalidate.
 * @return <code>true</code> if the account was created successfully.
 * @see module:models/CustomerModel~CustomerModel/createNewCustomer
 * @see module:models/CustomerModel~CustomerModel/setLogin
 */
CustomerModel.createAccount = function (email, password, form) {

    Transaction.begin();

    // Creates a new customer.
    var newCustomer, rememberMe;
    newCustomer = CustomerModel.createNewCustomer(email, password);
    if (newCustomer === null || newCustomer.object === null) {
        Transaction.rollback();
        form.invalidate();
        return false;
    }

    if (!Form.get('profile.customer').copyTo(newCustomer.object.profile)) {
        Transaction.rollback();
        form.invalidate();
        return false;
    }

    // Sets login and password for customer.
    if (!this.setLogin(newCustomer.object, email, password)) {
        Transaction.rollback();
        form.invalidate();
        return false;
    }

    Transaction.commit();

    Form.get('login').copyTo(newCustomer.object.profile.credentials);

    rememberMe = Form.get('profile.login.rememberme').value();
    
    /**
     * @type {dw.system.HookMgr}
     */
    const HookMgr = require('dw/system/HookMgr');
    var hookID = 'app.account.created';
    if (HookMgr.hasHook(hookID)) {
        HookMgr.callHook(
            hookID,
            hookID.slice(hookID.lastIndexOf('.') + 1),
            newCustomer.object
        );
    } else {
        require('dw/system/Logger').debug('no hook registered for {0}', hookID);
    }    

    // Logs the customer in.
    return Transaction.wrap(function () {
        return CustomerMgr.loginCustomer(email, password, rememberMe);
    });

};
```

**(3) OrderModel.js: Add below snippet to submit function**

```
    /**
     * @type {dw.system.HookMgr}
     */
    const HookMgr = require('dw/system/HookMgr');
    var hookID = 'app.order.created';
    if (HookMgr.hasHook(hookID)) {
        HookMgr.callHook(
            hookID,
            hookID.slice(hookID.lastIndexOf('.') + 1),
            order
        );
    } else {
        require('dw/system/Logger').debug('no hook registered for {0}', hookID);
    }
```

Here's complete submit function with the changes:

```
/**
 * Submits an order
 * @param order {dw.order.Order} The order object to be submitted.
 * @transactional
 * @return {Object} object If order cannot be placed, object.error is set to true. Ortherwise, object.order_created is true, and object.Order is set to the order.
 */
OrderModel.submit = function (order) {
    var Email = require('./EmailModel');
    var GiftCertificate = require('./GiftCertificateModel');
    try {
        Transaction.begin();
        placeOrder(order);

        // Creates gift certificates for all gift certificate line items in the order
        // and sends an email to the gift certificate receiver

        order.getGiftCertificateLineItems().toArray().map(function (lineItem) {
            return GiftCertificate.createGiftCertificateFromLineItem(lineItem, order.getOrderNo());
        }).forEach(GiftCertificate.sendGiftCertificateEmail);
        
        Transaction.commit();
    } catch (e) {
        Transaction.rollback();
        return {
            error: true,
            PlaceOrderError: new Status(Status.ERROR, 'confirm.error.technical')
        };
    }

    Email.sendMail({
        template: 'mail/orderconfirmation',
        recipient: order.getCustomerEmail(),
        subject: Resource.msg('order.orderconfirmation-email.001', 'order', null),
        context: {
            Order: order
        }
    });
 
    /**
     * @type {dw.system.HookMgr}
     */
    const HookMgr = require('dw/system/HookMgr');
    var hookID = 'app.order.created';
    if (HookMgr.hasHook(hookID)) {
        HookMgr.callHook(
            hookID,
            hookID.slice(hookID.lastIndexOf('.') + 1),
            order
        );
    } else {
        require('dw/system/Logger').debug('no hook registered for {0}', hookID);
    }
    
    return {
        Order: order,
        order_created: true
    };
}
```


## **Testing**

### **New Customer Registration**

To test integration with **Customer Registration **in Site Genesis (Usecase A: Customer registers on Commerce Cloud in [Service Cloud Connector: Release Notes](https://salesforce.quip.com/XtfuAneYCbjg)):

1.Open SiteGenesis storefront and navigate to Create Account to register a new test customer

[[images/installation-setup-31.png]]

2.Look up the newly created customer in Business Manager on your SFCC instance at Merchant Tools > Customers > Manage Customers. Locate “**Service Cloud Connector**” custom attribute section and confirm that all attributes are populated and  SyncStatus is set to “exported”. See below for an example.

[[images/installation-setup-32.png]]

3.In your Salesforce Instance (Lightning experience), use App Launcher to run “SCC Service Cloud” app you previously installed as part of the managed package.  Under “Contacts” , confirm that you can see the test Customer you just created on SFCC SiteGenesis storefront:
[[images/installation-setup-33.png]]

Note that Real-time sync will pull only basic data for the customer from Commerce Cloud to Service Cloud. Rest of the data (such as custom fields) will be synced via a job. Check under Setup→Apex Jobs to verify.

[[images/installation-setup-34.png]]

### **Order Sync for Registered Customer**

To test integration for registered customer order synchronization in SiteGenesis (Usecase E: Registered Customer places order on Commerce Cloud in [Service Cloud Connector: Release Notes](https://salesforce.quip.com/XtfuAneYCbjg)):

1.Make sure that you are logged in to the storefront as your test customer, created as part of testing Usecase A above. Place a product in basket, checkout and place an order in SiteGenesis storefront.

2.Navigate to this new order in Business Manager under Merchant Tools > Ordering > Orders.  Confirm that for that order, SF custom attributes are populated and Sync Status is set to “exported”. See below.

[[images/installation-setup-35.png]]

3.In your Salesforce Instance (Lightning experience), use App Launcher to run “SCC Service Cloud” app you previously installed as part of the managed package.  Under “Orders” , confirm that you can see the test order you just created on SFCC SiteGenesis storefront. Note that SFCC Order ID  is populated and the order is linked to the registered customer's Contact record. See below for an example.

[[images/installation-setup-36.png]]

Click “Details” tab of the Order record and scroll down to verify that complete SFCC order details are available as shown below:

[[images/installation-setup-37.png]]

### **Case Management: Create new Case**

In your storefront, open “Contact Us” page (in SiteGenesis, there's a link in footer menu) and fill out the form. On “Submit” of “Contact Us” form, a new Case  will be created in Service Cloud. For a registered customer, this Case will linked to their Contact record and data will be pre-populated. If valid Order number is provided on “Contact Us” form, respective Case will also be linked to Order record in Service Cloud.  For Guest user, a new  Case will be created with SFCC data populated, however no Contact record would be linked.

See below and example for a registered customer who placed an order:

[[images/installation-setup-38.png]]

Observe that a new Case has been created for this customer. Clicking on Case number will show details of the case, including data received from SFCC:

[[images/installation-setup-39.png]]

### **Order on behalf of (OOBO)**

To test OOBO, first create a Case for your customer in Service Cloud. From Case Details screen, click “Launch Shopping Cart” button (see below). It will open Commerce Cloud storefront with the customer already logged in. Please a new order and observe it getting synced with Service Cloud.

[[images/installation-setup-40.png]]