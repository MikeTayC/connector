# SC Connector : Troubleshooting

## Issues during SC Connector Package Install

**(1) Error: SortOrder must be in sequential order from 1. Contact.SCC_Contact_Duplicate_Rule: SortOrder must be in sequential order from 1.**

**Possible root cause: **You SF Development instance has more than one Contact Duplication rule(s).  Such rules create a conflict with the package configuration and need to be removed prior to package install. NOTE: v1.3  package requires max only one duplication rule because it tries to install in position #2.
This issue is can be related to SortOrder on contact duplication rule. Connector expects only one default contact duplicate rule to be in place.

We are trying to fix this next release.

**Resolution**: Remove conflicting Contact Duplication Rules and re-install the package.  Quick workaround for this issue is to keep only one contact duplicate rule in place and disable all remaining duplicate contact rules temporarily for installation. After installation you can enable them again. Other way is to make sure that SortOrder 2 is free and available in your org before installing connector package.

**(2) Salesforce REST API Error:**

`ERROR PipelineCallServlet|3662910|Sites-SiteGenesis-Site|Account-RegistrationForm|PipelineCall|enlMCaSa0i5pJdSpeDnxfp3RPL_-6Jd5-me6dAI9qccArwzmzgE3OHvwF4TAOJRms-O71ZD1i0Bsmdrwx0TNOQ== custom.service.servicecloud.rest.create.HEAD [] service=servicecloud.rest.create status=ERROR errorCode=403 errorMessage=[{"errorCode":"FORBIDDEN","message":"**You do not have access to the Apex class named: SCCCustomerRegistrationService"**}]`

**Possible root cause: **When installing the package, you  did not select the option to “Allow Access to All Users”.

**Resolution:** Add the Connector APEX classes to the profile created at step 1.1 of the installation guide. Without that, the user used by the API calls does not have access to the classes included in the SCC Connector package and cannot execute the related code on Salesforce. In below screenshot, the profile now has access to the required classes:


## Custom Logging Framework

[Content to be added]