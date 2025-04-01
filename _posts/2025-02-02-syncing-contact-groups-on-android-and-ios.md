---
layout: post
title:  "Syncing Contact Groups on Android and iOS with Nextcloud"
date:   2025-02-02 18:26:56 +0100
categories: posts
---

Using [DAVx⁵](https://www.davx5.com) to sync contacts on Android via CardDAV can be configured in a way that allows seamless group creation and editing on both Android and iOS.

<div style="display: flex; justify-content: center; align-items: center; gap: 10px; margin: 2rem 0;">
  <img src="/assets/images/android-contact-groups.png" 
       alt="Contact groups on Android" 
       style="max-width: 100%; height: auto; object-fit: contain; width: 350px;" />
  
  <img src="/assets/images/ios-contact-groups.png" 
       alt="Contact groups on iOS" 
       style="max-width: 100%; height: auto; object-fit: contain; width: 350px;" />
</div>

When setting up DAVx⁵ with Nextcloud, its [documentation](https://www.davx5.com/tested-with/nextcloud) recommends selecting _Groups are per-contact categories_ as the Contact group method. This is because the Nextcloud Contacts app only supports this option.
However, if compatibility with the Nextcloud Contacts app is not a priority, you can instead choose _Groups are separate vCards_
This method aligns with how iOS syncs contact groups over a CardDAV account.

<div style="display: flex; justify-content: center; align-items: center; gap: 10px; margin: 2rem 0;">
  <img src="/assets/images/dav5x-contact-group-method.png"
       alt="Contact group method on DAVx⁵" 
       style="max-width: 100%; height: auto; object-fit: contain; width: 350px;" />
</div>

With this configuration, any changes to group memberships or group creation/modification will now sync seamlessly between Android and iOS.
