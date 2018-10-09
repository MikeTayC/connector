# Service Cloud Connector: Release Notes

## **SCConnector v1.3/Feb 2018**

## Implemented Features

**IMPORTANT**: Every Customer List in SFCC will be mapped to an Account in Salesforce CRM. An Account is a mandatory object to reference all customer profiles ( a.k.a. Contacts in Service Cloud). At the time of SCC package installation Accounts will be created for each Customer List that exists on the respective SFCC instance.

### Customer Data Synchronization: Commerce Cloud ↔ Service Cloud 

**_Usecase A: Customer registers on Commerce Cloud._ **A respective Contact record will be created in Service Cloud and  mapped to respective Account ( equivalent of Customer List in SFCC) in Service Cloud. Two configurable customer data synchronization modes are available:

1. ***Real time data synchronization***: In this mode SFCC will call Service Cloud Contact API to push minimum required customer  profile information to create draft Contact record, when customer registers on Commerce  Cloud. Once draft Contact record is created, Service Cloud will call Commerce Cloud  OCAPI to pull customer details to synchronize all customer data and marks.  Contact status will be set as ‘synchronized’. This mode doesn’t stop customer  registration journey in case there is some integration issue between  Commerce Cloud and Service Cloud. In case of sync issue, customer profile  in Commerce Cloud stay in 'Created' mode and will be processed subsequently by asynchronous data synchronization job as explained below. 
2. ***[NEXT RELEASE] Asynchronous data synchronization***: In this mode Commerce Cloud doesn’t call any API in real time during customer journey on Commerce  Cloud. There is a job configured on Commerce Cloud, which calls Service  Cloud APIs asynchronously to push minimum customer profile data to create  draft Contact record. After that it’s same process of data synchronization  as in real time mode. This job is mandatory in both modes as it works as  backup in case real time integration leaves some customer profiles unsynchronized. 

**_Usecase B: Customer updates their  profile on Commerce Cloud_**. In this scenario, an API call is made to update Contact record status to ‘updated’  in Service Cloud, which in-turn triggers Usecase A data synchronization process  by pulling customer data using commerce cloud OCAPI and marking Contact  status to synchronized.

NOTE: Customer data can also be  modified within Service Cloud's Call Center, and will need to be synchronized  back to Commerce Cloud. This feature is planned in upcoming releases of  SCC.

_**Usecase D: Customer places guest order.**_  Contact record is mandatory in Service Cloud for SCC. When customer places a  guest order, a Contact record gets created if it doesn’t exist, then order  header is created and linked back to respective Contact. Email id  used as unique key for Contact in CRM. So every time a new Contact creation request  comes from any source, a data duplication rule will first check if Contact  already exists. If Contact already exists, then its CRM  reference is sent back. Guest order data synchronization is covered below  in order data synchronization section. 

_**Usecase E: Registered Customer places order on Commerce Cloud.**_  Contact record will already exist on Service Cloud for a Registered customer, created as part of Usecase A on Commerce Cloud. When this customer places an order on Commerce Cloud, order will be exported to Service Cloud and linked to the customer's record. SF will carry two order IDs: SF order ID and SFCC order ID.

_**Usecase F: New Case is created for Registered or Guest customer.**_
See “Case Management” section below for more information. “Contact Us” page in SFCC on “Submit” will create a new Case in Service Cloud. For registered customer, this Case will linked to their Contact record and data will be pre-populated. If valid Order number is provided on “Contact Us” form, respective Case will also be linked to Order record in Service Cloud.  For Guest user, a new  Case will be created with SFCC data populated, however no Contact record would be linked.

### Order Data Synchronization: Commerce Cloud ↔ Service Cloud

When customer places an order on Commerce Cloud through any selling channel, Order header gets pushed to Service Cloud. Order header contains order number, customer's CRM id (Salesforce Contact Id, if available), order total amount, order status, order SCC sync status.  There are two modes of Contact data sync: Real time or Asynchronous. In real time, Salesforce Contact ID gets updated immediately on registration where as in Async mode it gets updated when profile is exported to Salesforce.
Service Cloud does not keep order details as CRM is not order MDM.  Within Service Cloud, when business user clicks on order Id to open Order Details page, Service Cloud will call Commerce Cloud OCAPI to fetch order details on demand. 
Order in Service Cloud should always be linked to a Contact. When Order header is pushed to Service Cloud with a valid Contact it’s considered a registered customer's order, otherwise usually a guest order. There are some exceptions when guest order can be considered as registered order, the edge cases are detailed in guest order section below. It’s possible that customer is registered and has active account on Commerce Cloud, hence possibly has Contact record in Service Cloud but still order is pushed as guest order. There is custom field at order object level in Service Cloud which defines if order was pushed as registered or guest.


The following are processing workflows for the two order types: 

* **Registered customer order.** Usually when customer order header is pushed in Service Cloud with a valid Service Cloud Contact id, it’s considered a registered customer order. A field at order object level in service cloud declares if order is registered or guest customer order. When order header is pushed without a valid  Salesforce Contact id, it’s initially considered as guest order and Service Cloud will call Commerce Cloud OCAPI to pull order details to get customer information from order.  A post processor Apex script try to find Contact record in Service Cloud, using email Id from order details. If this post processor finds valid Contact and order details response from Commerce Cloud OCAPI shows that the customer, who placed the order, is marked as registered, then order is recorded as registered customer even though Salesforce Contact id was missing the the initial order header request. Latter scenario covers an edge case when something goes wrong on Commerce Cloud side while pushing order header to Service Cloud and Salesforce Contact id does not get sent correctly.
* **Guest customer order.** When customer order header data is pushed to Service Cloud and it doesn’t contain any Salesforce Contact id, the order will be considered a guest order. Order gets created with status 'Draft' and type 'guest'. Workflow configured on Service Cloud picks up Draft orders and will call Commerce Cloud OCAPI to fetch order details. From order details response this post processing workflow script will extract customer email id and at first step will lookup Contact in Service Cloud using email id. If Contact is available for this email id, then the order header status is changed to 'Created' and order gets linked with this Contact. If Contact is not found in Service Cloud, then a guest type contact is created and order header is linked with newly created Contact. Service Cloud does not create any profile back on Commerce Cloud for guest contacts. In future if same customer register on the storefront, then Contact is converted from guest to registered. All these changes are tracked in Contact object history. So using Contact history, business user can identify if when Contact converted from guest to registered. As explained above a registered Contact can always have guest orders linked to it because registered customer can always place guest order without logging in on Commerce Cloud.

### Case management

Every communication ( through contact us form, call to customer care, email to customer care) creates a Case linked to a Contact which belongs to customer. In scope of SCC we are covering only a scenario when customer communicates by submitting 'Contact Us' form on Commerce Cloud. In such case, if customer is logged in, then all customer metadata gets auto populated in 'Contact Us' form, for example name, email etc. When customer submits 'Contact Us' form, Commerce Cloud will call Service rest API to create a Case record in Service Cloud. In this service call request Commerce Cloud also sends linked Salesforce Contact id, if customer is logged in on Commerce Cloud. Salesforce Service Cloud creates Case record and links Case with Contact using provided Contact id. If Contact id was not sent in service call from Commerce Cloud, then Service Cloud will first lookup Contact using sent email id in case service call request. If Contact is available in Service Cloud, the Case gets linked with Contact otherwise new Contact gets created and Case gets linked to newly created Contact.

### Order on behalf of (OOB)

SCC provides capability to place order on behalf of customer in Commerce Cloud using Service Cloud console (Call Center). For initial version of SCC this capability is only available for a registered customer. OOB functionality will be added for guest order as well in upcoming releases. Service Cloud console provides 'Launch Shopping Cart' button on 'Case Details' screen of the console. When Call Center agent clicks on this button, it will open Commerce Cloud storefront with customer logged in on behalf of the working Call Center agent. Call Center agent can place an order from opened Commerce Cloud storefront. Such order will get recorded as 'on behalf of' order in Service Cloud under respective Contact record.

## Not Implemented in SCConnector v1.3

### Person Account Type

Service Cloud Connector version 1.3 doesn't support Service Cloud person account type and Commerce Cloud multi-site and multi-customer lists. Current release implementation will create a single default Account which gets mapped to Commerce Cloud customer list through configuration. All contacts (Customer Registration) will be created under this Account. Multi-site/multi-customer list support as well as person account support is planned for upcoming releases.

## SCConnector v1.4: Planned Features

### Customer Data Synchronization: Commerce Cloud ↔ Service Cloud 

**_Usecase A: Customer registers on Commerce Cloud._ **A respective Contact record will be created in Service Cloud and  mapped to respective Account ( equivalent of Customer List in SFCC) in Service Cloud. Two configurable customer data synchronization modes are available:

1. ***Real time data synchronization***: Implemented in v1.0, see above.
2. ***Asynchronous data synchronization***: In this mode Commerce Cloud doesn’t call any API in real time during customer journey on Commerce  Cloud. There is a job configured on Commerce Cloud, which calls Service  Cloud APIs asynchronously to push minimum customer profile data to create  draft Contact record. After that it’s same process of data synchronization  as in real time mode. This job is mandatory in both modes as it works as  backup in case real time integration leaves some customer profiles unsynchronized. 
3. ***Manual data sync:*** Agent will have an option to initiate data sync manually from Service Cloud.

_**Usecase C: Customer  opts-in for newsletter/marketing email.**_ In this scenario, Contact  record gets created in Service Cloud with type 'Opt-in'. For this usecase, we may have very limited  customer information, so Contact  will contain only customer's email with opt-in flag set.

### Working with Commerce Cloud Orders in Service Cloud

'CANCEL' order functionality for SFCC order will be added for Service Cloud agent to cancel a Commerce Cloud order, not yet exported to OMS.

### Multi-site/Multi-customer list support

A Service Cloud Account will be created for each Commerce Cloud customer list and Contacts will be created under respective Account (customer list). 

## SCConnector Roadmap: Planned Features

### Chat integration ( Live agent and Chat Bot)

Live agent provides chat capability which can be integrated with any website. As part of SCC integration a button for a chat pop up window is added. This button gets enabled on Commerce Cloud storefront when Call Center agent is logged in on Service Cloud console. When customer clicks on this button - if customer is logged in -  a chat window opens. For guest user it opens a form to fill in name and email id before opening chat window. Once chat is initiated, customer will wait till Call Center agent accepts chat request. When Call Center agent accepts chat request, a linked 'Contact Details' page automatically opens using sent email id. If Contact record is not found, a new Contact gets created on Service Cloud before starting chat. All chat history gets stored at Contact object for future reference.

### Data Synchronization: Service Cloud ↔ Commerce Cloud 

Adding bi-directional data sync capability.