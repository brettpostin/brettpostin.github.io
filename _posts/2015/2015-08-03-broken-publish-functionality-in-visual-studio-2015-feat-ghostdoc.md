---
layout: post
title:  "Broken “Publish” functionality in Visual Studio 2015 (feat. GhostDoc)"
date:   2015-08-03 20:56:17 +0000
categories: Tooling
tags: ghostdoc visual-studio
---

After installing Visual Studio 2015 along with all of my favourite extensions (including GhostDoc), an existing vNext project setup to publish to Azure could no longer be published.

Right clicking the project and choosing "Publish..." would seemingly do nothing.

Selecting the project and choosing _Build > Publish [ProjectName]_ resulted in an error dialog with the following message:

```
COM object that has been separated from its underlying RCW cannot be used.
```

After finding a related issue [here](https://github.com/aspnet/Tooling/issues/39) it turned out to be GhostDoc causing the problem. Uninstalling the extension fixed the issue.

The developers of GhostDoc appear to be aware of the issue in the above link, however the fix does not seem to have been rolled into the latest version (v4.9), unless of course it is a separate issue.

I'm looking forward to it being fixed so that I can reinstall what is usually a great extension.