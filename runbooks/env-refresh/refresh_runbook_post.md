### Runbook to record all production-specific metadata carried over upon refresh of a Salesforce environment.

1. Update the following Static Resources settings by replacing the production ids with the ids of the new Sandbox
    - LIVECHAT_FUNCTIONS_EDGE
    - LIVECHAT_FUNCTIONS_PSA
    - LIVECHAT_FUNCTIONS_SMS
    - LIVECHAT_FUNCTIONS_SSC
    - LIVECHAT_FUNCTIONS_SSC_Claimant
    - LIVECHAT_FUNCTIONS_PSA_EM

2. Change CORS setings to point to the GEICO URLs connected to the new sandbox. This step is needed only if GEICO is pointing an application environment to the new sandbox else remove the CORS settings, for example, in UAT
    - Change https://edgecustomer.geico.com to https://edgein-ut.geico.com
    - Change https://geicosurvey.force.com to https://uat-geicosurvey.cs211.force.com

    Other sandboxes have values that can be found here: https://salesforce.quip.com/x0AdAg3VNaJb

3. Change Session Settings to the URL of the new sandbox, for example,
    - Change https://geico-crm.secure.force.com to https://uat-geico-crm.cs211.force.com

4. Security Key Custom Metadata: Set proper key and IV for that sandbox. The values can be found here: https://salesforce.quip.com/x0AdAg3VNaJb

5. BOT
    - Turn Einstein Bots on
    - Deactivate each Bot
    - Reactivate current version for EDGE and Static (see docs/model_management.png)
   
   <mark>NOTE:</mark> Build model - will take several hours. You must do one at a time or the second one will kill the first.

6. Modify custom mulesoft endpoint label (this will go away later cause we stop using label and read dynamically via Custom Metadata per org)
    - DEV (DV1): https://esbgateway-np.geico.com/dv1/geico-crm-cs-eapi/api/v1/client/{ID} 
    - SIT (IN1): https://esbgateway-np.geico.com/in1/geico-crm-cs-eapi/api/v1/client/{ID}
    - UAT (UT1): https://esbgateway-np.geico.com/ut1/geico-crm-cs-eapi/api/v1/client/{ID}
    - LT (Load Test): https://esbgateway-lt.geico.com/lt/geico-crm-cs-eapi/api/v1/client/{ID}
    - PD (Production): https://esbgateway.geico.com/pd/geico-crm-cs-eapi/api/v1/client/{ID}

    <mark>NOTE:</mark> Only need this part for the label: ut1/geico-crm-cs-eapi/api/v1/client/. If not in list, use dv1/geico-crm-cs-eapi/api/v1/client/

7. Modify Auth Provider AND Custom Metadata with proper settings

    | Auth. Provider ID    | 0SO190000008Qky | 
    | Provider Type        | Custom |
    | Name                 | MicrosoftAzureClientCredentials |
    | URL Suffix           | MicrosoftAzureClientCredentials |
    | Plugin               | Azure_ClientCredentials_AuthProvider |
    | Callback URL         | /services/authcallback/MicrosoftAzureClientCredentials | 
    | Client ID            | fe7fb2a9-3a27-442c-8235-9ad4ad9d6468 |
    | Client Secret        | HFZnocLb8IM3xPSp00_Wn-DrSr6dgp~3B_ |
    | Scopes               | https://graph.microsoft.com/.default |      
    | Tenant ID            | 7389d8c0-3607-465c-a69f-7d4426502912 | 

    <mark>NOTE:</mark> updating the Auth Provider will automatically update the Custom Metadata

8. Modify Named Credential with proper non production URL
    - URL:	https://esbgateway-np.geico.com/. Then save and make sure that Authentication Status = Authenticated as     Azure Dummy User

9. Test Callout
    - Run this in Workbench or Dev Console:

        ```
        public class muletest extends MuleCalloutService {
            public muletest(){
                super();
            }
        } 
        string endpoint = '/ut1/geico-crm-cs-eapi/api/v1/client/';
        string clientid = 'ET000000004161348658';
  
        System.HttpResponse res = muletest.doGet(endpoint + clientid, null, null);
        system.debug('RESPONSE====> ' + res.getBody());
        
        ```
    The Client ID needn’t be valid for that endpoint but the callout should still succeed, simply telling you that the client id is not not valid as in: "message": "Individual Client not found or expired"

10. Splunk Connector: 
    
    Make sure Splunk can still connect - until the splunk connector is live (and event then post refresh steps will be needed):

    - Set up user w/ Integration user profile
    - Set up connected app https://test.salesforce.com/services/oauth2/token  as the callback for Splunk Connector
    - Give client id/sec to Andrea Page (POC for GEICO Splunk)
    - Set up with settings here https://docs.splunk.com/Documentation/AddOns/released/Salesforce/Setupv1
    - Relax IP Restrictions
    - Make sure Andrea is not locked out - token not needed
    - Event manager - enable events for monitoring (Streaming and storing) as in the screenshot docs/events.png

11. Single Sign-On: 
    
    We need the SAML Metadata file for the various environments and once we refresh we should be able to upload the metadata file. Also, under Single Sign-on

    - Set Federated Single Sign-On Using SAML
    - SAML Enabled =  TRUE

    Under My Domain:
    - Edit Authentication Configuration
    - Check box for “Azure AD” under Authentication Service

12. Mask sensitive data. This is applicable to Full and Partial Sandboxes (UAT, PerfTest, Training)

13. Disable or Reactivate Scheduled Apex Jobs

14. Reconfigure External Web Service calls for a non-production environment

15. Remove the email suffix for required users

16. Create any required users who don't exist in Production

17. Regenerate or deactivate Inbound Email Services

18. Delete / modify entries in Remote Site Settings if you don't want to perform certain callouts

19. Verify integrations are not pointing to Production

20. Update URLs in the Sandbox to point from Production to Sandbox

21. Configure linking with edgemock sandbox if it's requires QA testing













