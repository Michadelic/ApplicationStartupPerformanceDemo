# ApplicationStartupPerformanceDemo

A code repository for hosting demo code to optimize the startup performance of an app.

What's in here?
-----------------

In the master branch you can find:
* A suboptimal app build with SAPUI5
* If you throttle your browser or deploy it to a server with high latency, you will feel how slow it is
* You can apply the mechanismns shown in the blog below to boost the startup performance of the app and make it much faster
 
In the Result branch you can find:
* The optimized code of the app with all important steps from the blog posts applied
* It will be about 10x faster than the original code in the master branch for high-latency scenarios

Important: Set your browser language to en_US to avoid additional requests to i18n.properties files. No 404 response codes should appear in network trace.

Related Links
-------------

*Multi-Version-Availability*
* https://blogs.sap.com/2015/07/30/multi-version-availability-of-sapui5/
* https://sapui5.hana.ondemand.com/versionoverview.html

*Performance Optimizations – Best Practices Part I*
* https://blogs.sap.com/2016/10/29/sapui5-application-startup-performance-best-practices/

*Performance Optimizations - Best Practices Part II*
* https://blogs.sap.com/2016/11/19/sapui5-application-startup-performance-advanced-topics/

Contributions
-------------

All content is published under the Apache 2.0 license.
For more information check the LICENSE.txt file

If you spot any issues with the code or found a bug, please create an issue or a pull request and I will take care of it

Thank you,

Michael
SAPUI5 Development Team
