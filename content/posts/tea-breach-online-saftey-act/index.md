+++
date = "2025-08-01T20:18:00+01:00"
draft = false
title = "Tea App Leak Exposes Flaws in UK's Online Safety Act"
categories = ['Online Safety Act', 'Security', 'Data Breach']
+++

The UK's [Online Saftey Act](https://www.gov.uk/government/publications/online-safety-act-explainer/online-safety-act-explainer) passed into law in October 2023. The Act mandated that, effective 25th July 2025, platforms operating in the UK that host adult or harmful content must implement age verification systems, using facial scans or photographic identification as the means of verification.

Despite the Act's intentions, many criticise the legislation as opening the door to censorship. Others argue it is so easily avoidable that it is pointless; the downloading of a VPN enables individuals to completely bypass the verification process. Somewhat hilariously, it also seems images from video games such as [Garry's Mod](https://80.lv/articles/people-are-using-garry-s-mod-to-circumvent-the-uk-censorship-law) are sufficient to fool the age verification estimates.

The Act's stated goal, to protect children from harmful content is commendable. However, mandating verification through unknown third-party services poses a serious data privacy risk. For example, on the very day the mandating of age verification came into force in the UK, **Tea**, an application that had implemented its own photographic verification process, [suffered a major data leak.](https://zakforster.com/posts/tea-data-breach/)

In total, over 59 GB of data was leaked, including 72,000 images, 13,000 of which were selfies or photo IDs. This had major privacy implications for the individuals impacted by the leak; using the leaked dataset various malicious websites were created, ranging from an interactive dashboard enabling users to explore the locations at which the images were taken, to sites that ranked the individuals in the leak by their attractiveness.

#### Implications For The Online Saftey Act

The Act itself does not explicitly require that age verification providers enforce any specific security policies. There is no certified list of services that have undergone stringent security screening before they are allowed to handle our highly sensitive data. Many verification processes are outsourced to third parties, the services may be based in a juristiction that doesn't enforce robust data protection processes.

The responsibility to ensure that any third-party services comply with relevant data protection standards falls to the platform using the verification service. In the case of Tea, its privacy policy stated it would securely process and immediately delete the image following the verification process. The extract below is taken directly from Tea's [privacy policy:](https://www.teaforwomen.com/privacy)

> During the registration process, users are required to submit a selfie photo for verification purposes. This photo is securely processed and stored only temporarily and will be deleted immediately following the completion of the verification process.

If statements like these are all the platforms have to rely on, how can they be sure the services are secure?

Smaller platforms may not be able to absorb the cost of verification services or struggle to vet the provider sufficiently. This may lead to an increase in the number of platforms either leaving the UK market or deciding to 'roll their own verification service', the latter being more worrying given the increased prominence of vibe-coded platforms and the glaring security holes often left by AI generated code.

The Act places immense trust in third party age verification systems. We are seemingly supposed to trust that these services will safeguard our most sensitive personal data. But after incidents like the Tea app data leak, will you be trusting your private information to unvetted providers?
