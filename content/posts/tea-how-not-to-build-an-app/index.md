+++
date = "2025-07-30T22:24:00+01:00"
draft = true
title = "Tea App Data Breach: An Example of Gross Incompetence"
categories = ['Software Engineering', 'Security', 'Data Breach']
+++

In July 2025, **Tea** - an app that enables women to perform collective background checks on men by posting their prospective male dates personal information within the app and comment on them, without the individual's knowledge or concent suffered a major data leak. 

Irony of the situation aside, this was a major data leak that revealed the personal information of thousands of women. In total over 59 GB of data was leaked, including 72,000 images, of which 13,000 were selfies or photo ids, and 59,000 images from within the app. 

The initial breach was rapidly followed by a second, when a researcher at [404Media](https://www.404media.co/a-second-tea-breach-reveals-users-dms-about-abortions-and-cheating/) reported they were able to access more than 1.1 million private messages between users of the app, often discussing intimate topics,. This has led to the app [disabling DM functionality](https://www.404media.co/tea-app-turns-off-dms-after-exposing-messages-about-abortions-cheating/).

### How The Leaks Occured

The crux of the issue is user identification. To use the application a user must verify they are indeed a women by uploading a selfie or photo identification. 

The below extract is taken directly from Tea's [privacy policy](https://www.teaforwomen.com/privacy)

> During the registration process, users are required to submit a selfie photo for verification purposes. This photo is securely processed and stored only temporarily and will be deleted immediately following the completion of the verification process.