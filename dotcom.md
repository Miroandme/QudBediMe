# Making Changes to Dotcom

1. Content is stored in the DotCom github repository. For landing pages, all the assets and code are all in the directory "landing." Many of these landing pages have redirects in the root level .htaccess file, so that the "landing/" doesn't appear in the customer-facing URL.

2. The DEV and QA processes happen on the DotCom servers. So that would be wsdc.dev.sungevity.com, wsdc.uat.sungevity.com, etc.

3. DotCom is essentially static content, rather than data-driven, on the whole. Always lean towards simplicity where possible. The Careers page and About Sungevity page load content from other sites via php and iframes. The landing pages generally load funnels that are VisualForce and apex from our Salesforce site.

4. Releases are made on the standard release dates via the DotCom repository. This repository also undergoes a lot of hotfixes, because of the nature of marketing demands, legal needs, hot news items, etc.
