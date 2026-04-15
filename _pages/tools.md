---
layout: archive
title: "Integration workbench"
permalink: /tools/
author_profile: true
redirect_from:
  - /tools
  - /tools.html
---

{% include base_path %}

Hello!

As a consultant, I often run into gaps in the tools I work with daily. More often than not, building a quick script or a small web app turns out to be faster than waiting for someone else to address the problem. This page collects the tools and documentation I've produced along the way. 

1. 🔍 [SAP Integration Suite explorer](https://integration-report.streamlit.app). I use this tool whenever a quick as-is report of an Integration Suite tenant is required. You simply need to download the integration packages (not the artifacts!) as `.zip` files and drop them there. A `.csv` file that you can import into Excel will be automatically generated. Some cool stuff: a systematic ID is generated for each artifact, and Process Calls relating different iflows are identified and documented automatically. See [GitHub repo](https://github.com/malmriv/IntegrationSuiteExplorer) for documentation.

2. 🫧 [SOAP Blueprint](https://soapblueprint.streamlit.app). While SOAP as a protocol is very robust, designing SOAP APIs (especially in Integration Suite) can be difficult, as SAP does not currently provide any sort of framework for that. This tool is a graphical assistant that allows the user to create XML schemas, generate the corresponding WSDL, dummy messages and even a Postman/Bruno collection. See [GitHub repo](https://github.com/malmriv/SOAPBlueprint) and [blog post](https://blog.almag.ro/posts/2026/04/soap-blueprint-wsdl-generator-for-sap/).


