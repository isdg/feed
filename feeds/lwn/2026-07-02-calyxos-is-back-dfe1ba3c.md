---
title: CalyxOS is back
url: https://lwn.net/Articles/1081038/
published: "2026-07-02T20:58:45Z"
feed: lwn
guid: https://lwn.net/Articles/1081038/
---

# CalyxOS is back

In August 2025, the [CalyxOS](https://calyxos.org/) privacy-focused Android distribution [announced](https://calyxos.org/news/2025/08/01/a-letter-to-our-community/) that it was pausing all releases while it reworked its release process, security protocols, and changed its signing keys following the departure of one of its founders. The project has now [announced](https://calyxos.org/news/2026/07/01/calyxos-official-release-is-back/) that it is "officially back from the hiatus" with the 7.2.2.0 release.

> CalyxOS 7.2.2.0 is signed by us using [a new
> HSM-based, open-source signing solution](https://calyxos.org/news/2026/02/10/calyxos-hsm-signing/) we designed to enhance the security of the entire signing process, ensure redundancy, and remove single points of failure. You can verify CalyxOS 7.2.2.0 and future builds following [these
> instructions](https://calyxos.org/install/verify/). For anyone who is interested, the security audit report of the HSM provisioning ceremony script can be found [here](https://github.com/trailofbits/publications/blob/master/reviews/2026-01-calyx-hsm-provisioning-ceremony-scripts-securityreview.pdf).
>
> In addition, we also went through significant infrastructure improvements. In particular, we have set up a cleaner server structure to streamline each release. In response to Google's less frequent AOSP source code releases, our team developed scripts to reduce the overhead in applying monthly patches and updates. Please keep in mind, additional manual steps are still needed to compensate for AOSP changes, such as requesting and storing kernel sources with each update. Currently, our lead engineer is continuing the maintenance of the base device trees for both LineageOS and CalyxOS to bridge the gap created by the absence of Google Pixel device trees.
