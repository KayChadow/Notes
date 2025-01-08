# Server-Side Prototype Pollution Scanner
BApp

Although it's useful to try manually probing for sources in order to solidify your understanding of the vulnerability, this can be repetitive and time-consuming in practice. For this reason, we've created the Server-Side Prototype Pollution Scanner extension for Burp Suite, which enables you to automate this process. The basic workflow is as follows:

### Setup
1. Install the Server-Side Prototype Pollution Scanner extension from the BApp Store and make sure that it is enabled. For details on how to do this, see Installing extensions

### Find Targets
2. Explore the target website using Burp's browser to map as much of the content as possible and accumulate traffic in the proxy history.
3. In Burp, go to the Proxy > HTTP history tab.
4. Filter the list to show only in-scope items.
5. Select all items in the list.

### Scan
6. Right-click your selection and go to Extensions > Server-Side Prototype Pollution Scanner > Server-Side Prototype Pollution, then select one of the scanning techniques from the list.
7. When prompted, modify the attack configuration if required, then click OK to launch the scan.

### Results
8. If you're using Burp Suite Community Edition, you need to go to the Extensions > Installed tab, select the extension, then monitor its Output tab for any reported issues.