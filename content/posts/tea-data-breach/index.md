+++
date = "2025-07-30T22:24:00+01:00"
draft = false
title = "Tea App Data Breach: Learning From Gross Incompetence"
categories = ['Software Engineering', 'Security', 'Data Breach']
+++

In July 2025, **Tea** - an app that enables women to perform collective background checks on prospective dating partners, suffered a major data leak.

This was a significant data breach that exposed the personal information of thousands of women. In total, over 59 GB of data leaked, including 72,000 images, 13,000 of which were selfies or photo IDs. Many of the leaked images contained **Exif** data: metadata about the images, containing information such as the location of the person at the time the photo was taken.

Using the leaked data an interactive dashboard has since been produced, enabling users to explore the locations at which the images were taken. An additional site surfaced that exploited the leaked images for non-consensual attractiveness rankings, highlighting further privacy concerns.

The initial breach was rapidly followed by a second, when a researcher at [404Media](https://www.404media.co/a-second-tea-breach-reveals-users-dms-about-abortions-and-cheating/) reported that they were able to access more than 1.1 million private messages between users of the app, many of which included personally identifiable information. It was so bad that Tea [disabled DM functionality.](https://www.404media.co/tea-app-turns-off-dms-after-exposing-messages-about-abortions-cheating/)

### How the Leaks Occurred

A combination of user identification requirements, breaches of Tea's own data privacy policy, and a complete lack of understanding of the most basic security principles by the app developer caused this leak.

To use the application, a user must verify they are indeed a woman by uploading a selfie or photo identification. The extract below is taken directly from Tea's [privacy policy:](https://www.teaforwomen.com/privacy)

> During the registration process, users are required to submit a selfie photo for verification purposes. This photo is securely processed and stored only temporarily and will be deleted immediately following the completion of the verification process.

This wasn't true. The photos were neither processed securely nor deleted immediately. In fact, as stated by the anonymous 4Chan user who discovered the open database:

> Tea uploads all user verification submissions to a public Firebase bucket.

This leak wasn't a hack. By leaving their bucket publicly accessible and the contents unencrypted, Tea effectively left the front door open for anyone to walk in and take the data. All that was required was to access the Firebase URL.

Tea claimed this was a legacy bucket, with newer users handled differently. But that doesn’t absolve them of blame. There was a vast quantity of user data there unprotected that shouldn't have been.

The second leak is more interesting and appears to have been caused by a failure in the backend to verify that the user was authorised to access a specific resource. It seems that once logged in and authenticated, any user could issue API requests including message IDs or user identifiers that didn’t belong to them. In return, the backend would fetch conversations the user shouldn’t have had access to.

### Lessons From This

The security oversights demonstrated by the Tea team exhibited significant negligence and an inadequate understanding of essential security principles. Fortunately, preventing something like this in your own applications is relatively straightforward.

#### 1 - Enforce Policies & Encrypt Your Data

The bucket was misconfigured, left publicly accessible and unencrypted. To counter this, perform an audit of your cloud resources, ensuring each resource has the appropriate policies applied so that only your calling service can GET or PUT data. Ensure public access is disabled.

Using tools like [AWS KMS,](https://aws.amazon.com/kms/) you can encrypt the data within your buckets at rest, ensuring only authorised services or roles can encrypt or decrypt it. This adds another layer of protection - even if the bucket were mistakenly made public, an attacker wouldn’t necessarily be able to read the data without access to the decryption keys or the ability to edit the key policy.

KMS also supports audit logging via AWS CloudTrail, allowing you to track which principals access or attempt to decrypt data.

#### 2 - Implement Object-Level Authorisation

The mistake Tea made in the messages leak was that they only verified a request came from an authenticated user. Once you were in, you were free to look at anything.

Your backend should not only authenticate users but also verify that each request is authorised to access the specific resource being requested. Implement access control checks at the object level to ensure users can only view or modify data they own or are permitted to access.

#### 3 - Tie Up Loose Ends

Application code and cloud resources are liabilities, not assets. Every piece of infrastructure increases your potential attack surface and the probability of a breach. In Tea's case, the exposed Firebase bucket was described as a legacy resource, meaning it was no longer in active use for new users, but still contained data from past users. If a service or storage location is no longer part of your production environment, it should be decommissioned as soon as possible.

#### 4 - Storing Images? Strip Out Exif Data

If your application stores user uploaded images, it’s crucial to consider whether you have a valid reason to retain the associated Exif data. Exif data includes sensitive metadata such as GPS coordinates, timestamps, and device details.

Unless your application requires this information for its functionality, you should strip out all Exif data before storing or displaying the images. Otherwise, can expose your users to security risks, as was the case with the Tea userbase.
