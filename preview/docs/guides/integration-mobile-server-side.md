---
<<<<<<< HEAD
title: EUID Server-Side Integration Guide for Mobile
sidebar_label: Server-Side Integration for Mobile
pagination_label: EUID Server-Side Integration Guide for Mobile
description: Setting up a mobile integration with token generate and refresh both on the server side.
hide_table_of_contents: false
sidebar_position: 04
displayed_sidebar: docs
=======
title: UID2 Server-Side Integration Guide for Mobile
sidebar_label: Server-Side Integration for Mobile
pagination_label: UID2 Server-Side Integration Guide for Mobile
description: Setting up a mobile integration with token generate and refresh both on the server side.
hide_table_of_contents: false
sidebar_position: 04
displayed_sidebar: docs
---

import Link from '@docusaurus/Link';

<<<<<<< HEAD
# EUID Server-Side Integration Guide for Mobile

This guide is for mobile app publishers who want to manage the EUID token entirely on the server side:
=======
# UID2 Server-Side Integration Guide for Mobile

This guide is for mobile app publishers who want to manage the UID2 token entirely on the server side:
>>>>>>> 6cd49ff (Update from https://github.com/IABTechLab/uid2docs/commit/470efa727aa7532fad7cf4018b4debf05568c0f8)

- The token is generated on the server side.
- The token is refreshed as needed on the server side.

This setup requires that most of the code changes are done on the server side, with minimal changes in the mobile app.

One advantage of this approach is that if you're dealing with multiple platforms (Web / CTV / mobile), doing everything on the server side can reduce platform-specific efforts.

To implement using this approach, follow the instructions in [Publisher Integration Guide, Server-Side](integration-publisher-server-side.md).

<<<<<<< HEAD
If your server-side code is in Java or Python, you can use one of the EUID SDKs to make the HTTP requests to EUID, instead of writing your own source code. For details, refer to one of the following SDK guides:
=======
If your server-side code is in Java or Python, you can use one of the UID2 SDKs to make the HTTP requests to UID2, instead of writing your own source code. For details, refer to one of the following SDK guides:
>>>>>>> 6cd49ff (Update from https://github.com/IABTechLab/uid2docs/commit/470efa727aa7532fad7cf4018b4debf05568c0f8)

- [SDK for Java Reference Guide: Usage for Publishers](../sdks/sdk-ref-java.md#usage-for-publishers)
- [SDK for Python Reference Guide: Usage for Publishers](../sdks/sdk-ref-python.md#usage-for-publishers)
