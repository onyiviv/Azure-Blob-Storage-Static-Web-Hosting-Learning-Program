[Azure Blob Storage Static Web Hosting Learning Program.txt](https://github.com/user-attachments/files/28831204/Azure.Blob.Storage.Static.Web.Hosting.Learning.Program.txt)
Phase 1: Building the Core Storage Infrastructure
Before I can touch advanced networking (CDN/Custom Domains), I need to spin the storage back up.

Step 1: I create the Storage Account
Search for Storage accounts in the Azure Portal and click + Create.

Basics Tab:

Resource Group: Create a new one named rg-static-site-production.

Storage account name: I gave it a unique name (ovivstaticsite2026).
Region: I choose a primary region  (pacific india) 

Performance: Select Standard.

Redundancy: Select Locally-redundant storage (LRS).

Click Review + Create, then Create.

Step 2: Enable Static Website Hosting
Inside my storage account menu, I scroll down to Data management and click Static website.

Change the toggle to Enabled.

Set Index document name to index.html.(created on VS code)

Set Error document path to 404.html. (created on VS code)

Click Save.


Step 3: I uploaded my HTML Content
I used the exact same index.html and 404.html files I created earlier.

In the storage account menu under Data storage, click Containers.

Click into the newly generated $web container.

Click ↑ Upload, select your local index.html and 404.html files, and click Upload.

Step 4: Verify Direct Access
I open an incognito/private browser window.

Paste my Primary endpoint URL. Confirm the homepage loads.

Add /randomtext to the end of the URL to trigger and verify the 404.html page works.

https://ovivstaticsite2026.z29.web.core.windows.net/


Task 6: Integrating Azure CDN
Step 4: Create and Configure Azure CDN
I will wrap my storage account inside an Azure CDN profile to cache my website globally and enable secure HTTPS.
1. Provision the CDN Profile
In the Azure Portal search bar, type CDN profiles and select it.
Click + Create.
For Compare offerings, I choose Azure CDN Standard from Edgio or Azure CDN Standard from Microsoft (Microsoft is perfectly fine and simple for this). To manually change it, click Explore other offerings , then click Create.
Fill out the Basics:
Subscription: Select my student/active subscription.
Resource Group: Select my existing rg-static-site-production.
Name: Name your CDN profile (e.g., cdn-ovivstaticsite).
Region: pacific india (this is selected automatically).
Pricing tier: Standard.
Check the box that says Create a new CDN endpoint now.
Configure the Endpoint settings:
CDN endpoint name: Enter a unique name (this will form your new live URL, e.g., ovivsite). Your endpoint URL will look like ovivsite.azureedge.net.
Origin type: Select Static website. (Do not select "Storage"—selecting "Static website" ensures Azure talks to my web endpoint directly).
Origin hostname: Select my storage account's web endpoint from the dropdown list (it will look like ovivstaticsite2026.z13.web.core.windows.net).
Click Review + create, then click Create.


Step 5: Verify the CDN Deployment
CDNs take a few minutes to propagate files to edge servers around the world. Let's verify it works.

On my CDN Endpoint overview page, find the Endpoint hostname (e.g., [https://ovivsite.azureedge.net](https://ovivsite.azureedge.net)).

Wait about 3 to 5 minutes for the global cache to prime.

Open an incognito browser window and navigate to my new CDN endpoint URL.

Verify Success: I should see my website load seamlessly over HTTPS!

Verify the 404: Add /broken-link to my CDN URL to make sure the CDN correctly passes the error request back to my storage account's 404.html page.



🛑 Platform Restriction & Architectural Pivot Log
1. Issue Encountered: Standalone CDN Profile Deployment Failure
Action Attempted: Provisioning a standalone Azure Front Door and CDN profile (Standard Microsoft tier) mapped to the Azure Blob Storage primary web origin endpoint (ovivstaticsite2026.z13.web.core.windows.net).
Result: The deployment instantly failed at the Azure resource provider initialization stage (Microsoft.Cdn/Profiles).
Error Message Received: Free Trial and Student account is forbidden for Azure Frontdoor resources.
Root Cause: Microsoft imposes strict subscription-level resource constraints on standard Free Trial and Azure for Students subscriptions, completely blocking the deployment of enterprise networking resources like Azure Front Door and Classic CDN profiles to prevent potential bandwidth abuse.
2. Issue Encountered: Storage-Integrated Azure CDN Failure
Action Attempted: Bypassing the standalone Marketplace wizard by attempting to create an integrated CDN endpoint directly from the Security + networking > Azure CDN blade inside the Storage Account menu.
Result: The deployment failed with an identical BadRequest error.
Root Cause: Although accessed natively within the storage resource, the backend automation engine still triggers the legacy StorageIntegration-ClassicCdnDeployment script, which relies on the same restricted Microsoft.Cdn provider infrastructure, resulting in a firm policy block.
3. Engineering Solution & Architectural Pivot
To completely satisfy the task criteria—which strictly demands hosting content over an accelerated CDN endpoint, enforcing HTTPS security, and maintaining proper custom/error document routing—I successfully pivoted from Azure Blob Storage to Azure Static Web Apps (SWA).
Azure Static Web Apps natively couples the static application with an out-of-the-box global Content Delivery Network and automatic SSL/TLS certificate management. This alternative serverless architecture completely bypasses the Free Trial subscription block while achieving identical global performance, security, and scalability metrics.
4. Implementation Workflow (Local Code to Azure SWA via GitHub)
To execute this pivot, I shifted from manual portal uploads to a modern DevOps CI/CD pipeline using Visual Studio Code and GitHub:
Local Workspace Preparation: Inside Visual Studio Code, I verified the local directory containing my project assets (index.html and 404.html).
Git Initialization: Using the integrated terminal in VS Code, I initialized a local Git tracking repository and took a version control snapshot using:
Bash
git init
git add .
git commit -m "Initial commit of static site code"
* **GitHub Remote Syncing:** I created a clean, public repository on GitHub named `azure-static-web-hosting`. I then linked my local VS Code environment to the remote cloud repository and securely pushed the code up using:
  ```bash
  git remote add origin https://github.com/onyiviv/azure-static-web-hosting.git
  git branch -M main
  git push -u origin main
Azure Static Web App Deployment: Back in the Azure Portal, I provisioned an Azure Static Web App under the completely free hosting tier. I authenticated my GitHub account (onyiviv), pointed the source deployment engine to the azure-static-web-hosting repository, and set the build application location root to /.
Upon final validation and creation, Azure automatically established a permanent deployment pipeline. This pulls the source files directly from GitHub and distributes them instantaneously across Microsoft’s global CDN edge network with pre-configured HTTPS.





Summary Markdown
# Enterprise Cloud Architecture: Global Static Web Hosting on Azure
## 🌐 Live Deployment* **Production URL:** [https://nice-forest-03fb21300.7.azurestaticapps.net](https://nice-forest-03fb21300.7.azurestaticapps.net)* **Hosting Paradigm:** Serverless Content Delivery Network (CDN) Edge Architecture

---
## 📋 Project Overview & Architectural Goals
This project demonstrates the transition from traditional, infrastructure-heavy web hosting (Virtual Machines running Nginx or Apache) to a modern, zero-maintenance serverless hosting strategy. The primary objective was to deploy a highly available, globally scalable frontend workspace while optimizing cost, load times, and operational complexity.
### Key Deliverables Achieved:1. Automated deployment pipeline via continuous integration/continuous deployment (CI/CD).2. Global content distribution via an edge-cached network layout.3. Strict end-to-end HTTPS encryption management.4. Custom error route handling via a dedicated fallback interface.

---
## 🛠️ Implementation & Step-by-Step Configuration
### Phase 1: Local Workspace Development & Git Control1. Constructed core frontend components (`index.html` and `404.html`) locally.2. Initialized version control boundaries within the working directory using the Git CLI:
   ```bash
   git init
   git add .
   git commit -m "Initial commit of static site code"
Established a remote origin pathway pointing to a public GitHub repository, setting the workspace branch structure to main:
Bash
git remote add origin [https://github.com/onyiviv/azure-static-web-hosting.git](https://github.com/onyiviv/azure-static-web-hosting.git)
git branch -M main
git push -u origin main
Phase 2: Production Deployment via Azure
Provisioned an Azure Static Web App (SWA) resource container within the rg-static-site-production Resource Group wrapper.
Authenticated the Azure Deployment Engine against the GitHub repository security context.
Established an automated build configuration targeting the root directory (/) as the application distribution boundary.
Executed the automated workflow pipeline, mapping code modifications directly from the repository stream to Microsoft's globally distributed cloud framework.
🛑 Platform Restriction & Architectural Pivot Log
1. Issue Encountered: Standalone CDN Profile Deployment Failure
Action Attempted: Provisioning a standalone Azure Front Door and CDN profile (Standard Microsoft tier) mapped to an Azure Blob Storage primary web origin endpoint.
Result: The deployment instantly failed at the Azure resource provider initialization stage (Microsoft.Cdn/Profiles).
Error Message Received: Free Trial and Student account is forbidden for Azure Frontdoor resources.
Root Cause: Microsoft imposes strict subscription-level resource constraints on standard Free Trial and Azure for Students subscriptions, completely blocking the deployment of enterprise networking resources like Azure Front Door and Classic CDN profiles to prevent potential bandwidth abuse.
2. Issue Encountered: Storage-Integrated Azure CDN Failure
Action Attempted: Bypassing the standalone Marketplace wizard by attempting to create an integrated CDN endpoint directly from the Security + networking > Azure CDN blade inside the Storage Account menu.
Result: The deployment failed with an identical BadRequest error.
Root Cause: Although accessed natively within the storage resource, the backend automation engine still triggers the legacy StorageIntegration-ClassicCdnDeployment script, which relies on the same restricted Microsoft.Cdn provider infrastructure, resulting in a firm policy block.

3. Engineering Solution & Architectural Pivot
To completely satisfy the evaluator’s grading criteria—which strictly demands hosting content over an accelerated CDN endpoint, enforcing HTTPS security, and maintaining proper custom/error document routing—I successfully pivoted from Azure Blob Storage to Azure Static Web Apps (SWA).
Azure Static Web Apps natively couples the static application with an out-of-the-box global Content Delivery Network and automatic SSL/TLS certificate management. This alternative serverless architecture completely bypasses the Free Trial subscription block while achieving identical global performance, security, and scalability metrics.
📊 Performance Optimization & Edge Metrics
To evaluate the caching efficiency and transmission speeds of the global content delivery layer, performance benchmarks were recorded utilizing Google Lighthouse telemetry.
Performance Score: 99/100 (desktop)
78/100 (mobile)
Time to Interactive (TTI): < 0.5s
Security Validation: Enforced TLS 1.3 / HTTPS encryption.The high metrics confirm the advantages of serverless edge routing over a standard VM web origin. Static assets are cached directly at regional points of presence (PoPs), mitigating database latency, eliminating container spin-up delays, and dramatically minimizing geographical physical distance latency.
📸 Deployment Artifact Evidence
Screenshots of every activities
---
