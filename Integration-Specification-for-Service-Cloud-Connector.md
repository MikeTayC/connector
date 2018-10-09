[[images/integration-spec-header.png]]
# Integration Specification for SC Connector

* * *

## Service Cloud Connector: Overview

The Service Cloud Connector (SCC) is a reusable code asset to support the enablement and acceleration of specific integration use cases between Commerce Cloud and Service Cloud. SiteGenesis is used as a reference storefront for SCC cartridge integration.

SCV (Single view of customer) is integral part of omni-channel platform. For B2C Commerce Cloud platform, SCC will provide a capability to leverage Salesforce CRM as MDM (Master Data Management) destination of customer data by allowing to centralize all customer data flowing from various data capture sources. SCC will push all customer data captured via different SFCC channels to Salesforce CRM. Following are known usecases to capture customer data on  SFCC eCommerce platform:

1. Customer registers on SFCC storefront or on mobile app integrated with SFCC platform.
2. Customer account (profile) is created by store agent using in-store app (EA).
3. Customer account is created by call Center agent working on behalf of customer.
4. Customer opts-in for marketing newsletter via web storefront, mobile app, communities, social channels.
5. Customer places guest order on web storefront, mobile, through in-store app or by calling call Center.

SC Connector cartridge will have new functionality added to it in an incremental manner. 


## Integration Architecture Diagram


[[images/integration-architecture-diagram.png]]
## Implementation Approach In a Nutshell

**SFSC Objects synchronized with Commerce Cloud:**

* Contact/Customer
* Case
* Order
* Activity/Task

**Synchronization modes:**

* Synchronous: REST call (e.g. at customer's registration)
* Asynchronous:  Job calls, Bulk API ( e.g. action is customer post-registration)

**Data sync**

Customer and order sync to Service Cloud via jobs


## **Service Cloud Connector v1.0 - Implemented Requirements**

**IMPORTANT**: Every Customer List in SFCC will be mapped to an Account in Salesforce CRM. An Account is a mandatory object to reference all customer profiles ( a.k.a. Contacts in Service Cloud). At the time of SCC package installation Accounts will be created for each Customer List that exists on the respective SFCC instance.

## Customer Data Synchronization: Commerce Cloud ↔ Service Cloud 

**_Usecase A: Customer registers on Commerce Cloud._ **A respective Contact record will be created in Service Cloud and  mapped to respective Account ( equivalent of Customer List in SFCC) in Service Cloud. Two configurable customer data synchronization modes are available:

1. ***Real time data synchronization***: In this mode SFCC will call Service Cloud Contact API to push minimum required customer  profile information to create draft Contact record, when customer registers on Commerce  Cloud. Once draft Contact record is created, Service Cloud will call Commerce Cloud OCAPI to pull customer details to synchronize all customer data and marks.  Contact status will be set as ‘synchronized’. This mode doesn’t stop customer  registration journey in case there is some integration issue between  Commerce Cloud and Service Cloud. In case of sync issue, customer profile  in Commerce Cloud stay in 'Created' mode and will be processed subsequently by asynchronous data synchronization job as explained below. 
2. ***Asynchronous data synchronization***: In this mode Commerce Cloud doesn’t call any API in real time during customer journey on Commerce  Cloud. There is a job configured on Commerce Cloud, which calls Service  Cloud APIs asynchronously to push minimum customer profile data to create  draft Contact record. After that it’s same process of data synchronization  as in real time mode. This job is mandatory in both modes as it works as  backup in case real time integration leaves some customer profiles unsynchronized. 

**_Usecase B: Customer updates their  profile on Commerce Cloud_**. In this scenario, an API call is made to update Contact record status to ‘updated’  in Service Cloud, which in-turn triggers Usecase A data synchronization process  by pulling customer data using commerce cloud OCAPI and marking Contact  status to synchronized.

NOTE: Customer data can also be  modified within Service Cloud's Call Center, and will need to be synchronized  back to Commerce Cloud. This feature is planned in upcoming releases of  SCC.

_**Usecase C: Customer  opts-in for newsletter/marketing email.**_ In this scenario, Contact  record gets created in Service Cloud with type 'Opt-in'. For this usecase, we may have very limited  customer information, so Contact  will contain only customer's email with opt-in flag set.

_**Usecase D: Customer places guest order.**_  Contact record is mandatory in Service Cloud for SCC. When customer places a  guest order, a Contact record gets created if it doesn’t exist, then order  header is created and linked back to respective Contact. Email id  used as unique key for Contact in CRM. So every time a new Contact creation request  comes from any source, a data duplication rule will first check if Contact  already exists. If Contact already exists, then its CRM  reference is sent back. Guest order data synchronization is covered below  in order data synchronization section. 

## Order Data Synchronization: Commerce Cloud ↔ Service Cloud

When customer places an order on Commerce Cloud through any selling channel, Order header gets pushed to Service Cloud. Order header contains order number, customer's CRM id (Salesforce Contact Id, if available), order total amount, order status, order SCC sync status. Service Cloud does not keep order details as CRM is not order MDM.  Within Service Cloud, when business user clicks on order Id to open Order Details page, Service Cloud will call Commerce Cloud OCAPI to fetch order details on demand. 
Order in Service Cloud should always be linked to a Contact. When Order header is pushed to Service Cloud with a valid Contact it’s considered a registered customer's order, otherwise usually a guest order. There are some exceptions when guest order can be considered as registered order, the edge cases are detailed in guest order section below. It’s possible that customer is registered and has active account on Commerce Cloud, hence possibly has Contact record in Service Cloud but still order is pushed as guest order. There is custom field at order object level in Service Cloud which defines if order was pushed as registered or guest.

The following are processing workflows for the two order types: 

* **Registered customer order.** Usually when customer order header is pushed in Service Cloud with a valid Service Cloud Contact id, it’s considered a registered customer order. A field at order object level in service cloud declares if order is registered or guest customer order. When order header is pushed without a valid  Salesforce Contact id, it’s initially considered as guest order and Service Cloud will call Commerce Cloud OCAPI to pull order details to get customer information from order.  A post processor Apex script try to find Contact record in Service Cloud, using email Id from order details. If this post processor finds valid Contact and order details response from Commerce Cloud OCAPI shows that the customer, who placed the order, is marked as registered, then order is recorded as registered customer even though Salesforce Contact id was missing the the initial order header request. Latter scenario covers an edge case when something goes wrong on Commerce Cloud side while pushing order header to Service Cloud and Salesforce Contact id does not get sent correctly.
* **Guest customer order.** When customer order header data is pushed to Service Cloud and it doesn’t contain any Salesforce Contact id, the order will be considered a guest order. Order gets created with status 'Draft' and type 'guest'. Workflow configured on Service Cloud picks up Draft orders and will call Commerce Cloud OCAPI to fetch order details. From order details response this post processing workflow script will extract customer email id and at first step will lookup Contact in Service Cloud using email id. If Contact is available for this email id, then the order header status is changed to 'Created' and order gets linked with this Contact. If Contact is not found in Service Cloud, then a guest type contact is created and order header is linked with newly created Contact. Service Cloud does not create any profile back on Commerce Cloud for guest contacts. In future if same customer register on the storefront, then Contact is converted from guest to registered. All these changes are tracked in Contact object history. So using Contact history, business user can identify if when Contact converted from guest to registered. As explained above a registered Contact can always have guest orders linked to it because registered customer can always place guest order without logging in on Commerce Cloud.

## Case management

Every communication ( through contact us form, call to customer care, email to customer care) creates a Case linked to a Contact which belongs to customer. In scope of SCC we are covering only a scenario when customer communicates by submitting 'Contact Us' form on Commerce Cloud. In such case, if customer is logged in, then all customer metadata gets auto populated in 'Contact Us' form, for example name, email etc. When customer submits 'Contact Us' form, Commerce Cloud will call Service rest API to create a Case record in Service Cloud. In this service call request Commerce Cloud also sends linked Salesforce Contact id, if customer is logged in on Commerce Cloud. Salesforce Service Cloud creates Case record and links Case with Contact using provided Contact id. If Contact id was not sent in service call from Commerce Cloud, then Service Cloud will first lookup Contact using sent email id in case service call request. If Contact is available in Service Cloud, the Case gets linked with Contact otherwise new Contact gets created and Case gets linked to newly created Contact.

## Order on behalf of (OOB)

SCC provides capability to place order on behalf of customer in Commerce Cloud using Service Cloud console (Call Center). For initial version of SCC this capability is only available for a registered customer. OOB functionality will be added for guest order as well in upcoming releases. Service Cloud console provides 'Place Order' button on 'Contact Details' page. When Call Center agent clicks on this button, it will open Commerce Cloud storefront with customer logged in on behalf of the working Call Center agent. Call Center agent can place an order from opened Commerce Cloud storefront. Such order will get recorded as 'on behalf of' order in Service Cloud under respective Contact record.

## Chat integration ( Live agent)

Live agent provides chat capability which can be integrated with any website. As part of SCC integration a button for a chat pop up window is added. This button gets enabled on Commerce Cloud storefront when Call Center agent is logged in on Service Cloud console. When customer clicks on this button - if customer is logged in -  a chat window opens. For guest user it opens a form to fill in name and email id before opening chat window. Once chat is initiated, customer will wait till Call Center agent accepts chat request. When Call Center agent accepts chat request, a linked 'Contact Details' page automatically opens using sent email id. If Contact record is not found, a new Contact gets created on Service Cloud before starting chat. All chat history gets stored at Contact object for future reference.

 NOTE: Chat bot integration is planned for a future release. 
 

## Service Cloud Connector - Solution Package

SCC solution is comprised of the following assets:

1. **A managed Salesforce package for Service Cloud. **This package contains all required Salesforce CRM components required to integrate with SFCC. The package comes fully configurable at time of installations with capabilities to change configuration post installation, with new capabilities added with upcoming releases. SCC package is currently hosted on Salesforce internal demo org. It is planned to publish it on Salesforce marketplace for wider community access. 
2. **SFCC cartridges.** SCC comes with two commerce cloud cartridge (one integration cartridge for integration connections with Salesforce CRM and second commerce cloud storefront extension cartridge containing hooks to call integration cartridge to synchronise data with Salesforce CRM.
3.  **SFCC configuration metadata.** A collection of metadata configuration XML files is provided and required to run SCC. It contains various SFCC configurations, settings and object extensions. 



## Implementation Details

### Storefront integration touch-points

The following SiteGenesis storefront pages will trigger integration usecases in SC Connector:

* Contact Us
* Customer Registration
* Checkout - Place Order

Please, see [Service Cloud Connector: Release Notes](https://salesforce.quip.com/XtfuAneYCbjg) for details of implemented usecases in each SCConnector release.

### General Implementation Approach

* Use Site Prefs to toggle sync/async mode for the sc-connector cartridge
* Auth token to create Contact in SF, pass minimum info for better performance
* SFSC will call OCAPI to sync more data when someone accesses object data (e.g. opens Contact)
* Apex triggers on SC-side

## Setup and Configuration

To run Service Cloud Connector,  we need at least one Commerce Cloud instance and one Service Cloud org. Installation is basically a 3-step process:

**Step 1:** Install SCC Salesforce package on your Service Cloud org. For more information on SCC Salesforce package installations please refer to [SC Connector: Installation Guide](https://salesforce.quip.com/2STQAtFcmw81) 

**Step 2:** Import all provided SFCC configuration metadata on your Commerce Cloud instance. 

**Step 3**: Upload commerce cloud cartridges and add them in respective site class path. For more information on SCC cartridge installations and metadata setup, please refer to [SC Connector: Installation Guide](https://salesforce.quip.com/2STQAtFcmw81) 


**Integration cartridge: [int_service_cloud](https://bitbucket.org/demandware/service-cloud-connector/src/14536fb6d82d6c7a14a0d9a089f41d72c69e0ba4/cartridges/int_service_cloud/?at=master)**

See below for the *int_service_cloud* cartridge file structure.

[[images/integration-folder-structure.png]]

## Customer Sync - Implementation Details

[More details are coming soon]

### REST APIS

### Authentication

SC uses oAuth for authentication, which returns a token upon successful auth.

### Response Handling

### OCAPI Hooks



## Order Sync - Implementation Details

[More details are coming soon]

## Commerce Cloud Metadata

### Services metadata

**File:** [service-cloud-connector](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93?at=master) / [sites](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93/sites/?at=master) / [site_template](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93/sites/site_template/?at=master) / services.xml

**XML:**

```
<?xml version="1.0" encoding="UTF-8"?>
<services xmlns="http://www.demandware.com/xml/impex/services/2014-09-26">
    <service-credential service-credential-id="servicecloud.rest.auth-SiteGenesis">
        <url>https://login.salesforce.com/services/oauth2/token</url>
        <user-id>API User ID</user-id>
        <password masked="true">********</password>
    </service-credential>

    <service-profile service-profile-id="profile-servicecloud.rest">
        <timeout-millis>10000</timeout-millis>
        <rate-limit-enabled>false</rate-limit-enabled>
        <rate-limit-calls>0</rate-limit-calls>
        <rate-limit-millis>0</rate-limit-millis>
        <cb-enabled>true</cb-enabled>
        <cb-calls>2</cb-calls>
        <cb-millis>100</cb-millis>
    </service-profile>

    <service service-id="servicecloud.rest.auth">
        <service-type>HTTP</service-type>
        <enabled>true</enabled>
        <log-prefix>servicecloud-rest-auth</log-prefix>
        <comm-log-enabled>true</comm-log-enabled>
        <mock-mode-enabled>false</mock-mode-enabled>
        <profile-id>profile-servicecloud.rest</profile-id>
        <credential-id>servicecloud.rest.auth-SiteGenesis</credential-id>
    </service>

    <service service-id="servicecloud.rest.create">
        <service-type>HTTP</service-type>
        <enabled>true</enabled>
        <log-prefix>servicecloud-rest-create</log-prefix>
        <comm-log-enabled>true</comm-log-enabled>
        <mock-mode-enabled>false</mock-mode-enabled>
        <profile-id/>
        <credential-id/>
    </service>

    <service service-id="servicecloud.rest.get">
        <service-type>HTTP</service-type>
        <enabled>true</enabled>
        <log-prefix>servicecloud-rest-get</log-prefix>
        <comm-log-enabled>true</comm-log-enabled>
        <mock-mode-enabled>false</mock-mode-enabled>
        <profile-id/>
        <credential-id/>
    </service>

</services>
```



### Commerce Cloud Site Preferences & Object Extensions

**Files:** 

* [service-cloud-connector](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93?at=master) / [sites](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93/sites/?at=master) / [site_template](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93/sites/site_template/?at=master) / [meta](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93/sites/site_template/meta/?at=master) / system-objecttype-extensions.xml
* service-cloud-connector / [sites](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93/sites/?at=master) / [site_template](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93/sites/site_template/?at=master) / [meta](https://bitbucket.org/demandware/service-cloud-connector/src/0174e6e03b93/sites/site_template/meta/?at=master) / custom-objecttype-definitions.xml

Note that all passwords will need to be manually set. Password values will not be imported.

**XML for System Object ****extensions:**

```
<?xml version="1.0" encoding="UTF-8"?>
<metadata xmlns="http://www.demandware.com/xml/impex/metadata/2006-10-31">
    <type-extension type-id="Profile">
        <custom-attribute-definitions>
            <attribute-definition attribute-id="sscSyncResponseText">
                <display-name xml:lang="x-default">Salesforce Sync Response Text</display-name>
                <type>set-of-string</type>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
            </attribute-definition>
            <attribute-definition attribute-id="sscSyncStatus">
                <display-name xml:lang="x-default">Salesforce Service Cloud SyncStatus</display-name>
                <type>enum-of-string</type>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
                <value-definitions>
                    <value-definition>
                        <display xml:lang="x-default">Account Exported</display>
                        <value>exported</value>
                    </value-definition>
                    <value-definition>
                        <display xml:lang="x-default">Account Created</display>
                        <value>created</value>
                    </value-definition>
                    <value-definition>
                        <display xml:lang="x-default">Account Updated</display>
                        <value>updated</value>
                    </value-definition>
                </value-definitions>
            </attribute-definition>
            <attribute-definition attribute-id="sscid">
                <display-name xml:lang="x-default">Salesforce Contact Id</display-name>
                <description xml:lang="x-default">Salesforce Contact Id</description>
                <type>string</type>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
                <min-length>0</min-length>
            </attribute-definition>
        </custom-attribute-definitions>
        <group-definitions>
            <attribute-group group-id="ServiceCloudConnector">
                <display-name xml:lang="x-default">Service Cloud Connector</display-name>
                <attribute attribute-id="sscSyncStatus"/>
                <attribute attribute-id="sscid"/>
                <attribute attribute-id="sscSyncResponseText"/>
            </attribute-group>
        </group-definitions>
    </type-extension>

    <type-extension type-id="Order">
        <custom-attribute-definitions>
            <attribute-definition attribute-id="sscSyncResponseText">
                <display-name xml:lang="x-default">Salesforce Sync Response Text</display-name>
                <description xml:lang="x-default">Salesforce Sync Response Text</description>
                <type>set-of-string</type>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
            </attribute-definition>
            <attribute-definition attribute-id="sscSyncStatus">
                <display-name xml:lang="x-default">Salesforce Service Cloud SyncStatus</display-name>
                <type>enum-of-string</type>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
                <value-definitions>
                    <value-definition>
                        <display xml:lang="x-default">Order Created</display>
                        <value>created</value>
                    </value-definition>
                    <value-definition>
                        <display xml:lang="x-default">Order Exported</display>
                        <value>exported</value>
                    </value-definition>
                    <value-definition>
                        <display xml:lang="x-default">Order Updated</display>
                        <value>updated</value>
                    </value-definition>
                </value-definitions>
            </attribute-definition>
            <attribute-definition attribute-id="sscid">
                <display-name xml:lang="x-default">Salesforce Order Id</display-name>
                <description xml:lang="x-default">Salesforce Order Id</description>
                <type>string</type>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
                <min-length>0</min-length>
            </attribute-definition>
        </custom-attribute-definitions>
        <group-definitions>
            <attribute-group group-id="general">
                <display-name xml:lang="x-default">General</display-name>
                <attribute attribute-id="notesAndHistory"/>
                <attribute attribute-id="sscSyncStatus"/>
                <attribute attribute-id="sscSyncResponseText"/>
                <attribute attribute-id="sscid"/>
            </attribute-group>
        </group-definitions>
    </type-extension>

    <type-extension type-id="SitePreferences">
        <custom-attribute-definitions>
            <attribute-definition attribute-id="sscIsAsync">
                <display-name xml:lang="x-default">Is async mode for Service Cloud?</display-name>
                <description xml:lang="x-default">When set, Service Cloud Connector will leverage asynchronous mode if data sync, via jobs</description>
                <type>boolean</type>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
            </attribute-definition>
            <attribute-definition attribute-id="sscUsername">
                <display-name xml:lang="x-default">Salesforce Connected App Username</display-name>
                <description xml:lang="x-default">Salesforce Connected App Username</description>
                <type>string</type>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
                <min-length>0</min-length>
            </attribute-definition>
            <attribute-definition attribute-id="sscPassword">
                <display-name xml:lang="x-default">Salesforce Connected App Password</display-name>
                <description xml:lang="x-default">Salesforce Connected App Password</description>
                <type>string</type>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
                <min-length>0</min-length>
            </attribute-definition>
        </custom-attribute-definitions>
         <group-definitions>

            <attribute-group group-id="ServiceCloudConnector">
                <display-name xml:lang="x-default">Service Cloud Connector</display-name>
                <attribute attribute-id="sscIsAsync"/>
                <attribute attribute-id="sscUsername"/>
                <attribute attribute-id="sscPassword"/>
            </attribute-group>

        </group-definitions>
    </type-extension>
</metadata>
```


**XML for Custom Object ****definitions:**

```
<?xml version="1.0" encoding="UTF-8"?>
<metadata xmlns="http://www.demandware.com/xml/impex/metadata/2006-10-31">
    <custom-type type-id="ServiceCloudAuthToken">
        <display-name xml:lang="x-default">ServiceCloudAuthToken</display-name>
        <description xml:lang="x-default">ServiceCloudAuthToken</description>
        <staging-mode>no-staging</staging-mode>
        <storage-scope>organization</storage-scope>
        <key-definition attribute-id="siteID">
            <type>string</type>
            <min-length>0</min-length>
        </key-definition>
        <attribute-definitions>
            <attribute-definition attribute-id="token">
                <display-name xml:lang="x-default">Token</display-name>
                <type>text</type>
                <localizable-flag>false</localizable-flag>
                <mandatory-flag>false</mandatory-flag>
                <externally-managed-flag>false</externally-managed-flag>
            </attribute-definition>
        </attribute-definitions>
        <group-definitions>
            <attribute-group group-id="SSCAuthToken">
                <display-name xml:lang="x-default">SSCAuthToken</display-name>
                <attribute attribute-id="lastModified" system="true"/>
                <attribute attribute-id="token"/>
                <attribute attribute-id="creationDate" system="true"/>
                <attribute attribute-id="siteID"/>
            </attribute-group>
        </group-definitions>
    </custom-type>
</metadata>
```