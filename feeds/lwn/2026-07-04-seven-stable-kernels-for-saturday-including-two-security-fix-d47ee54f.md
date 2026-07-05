---
title: Seven stable kernels for Saturday including two security fixes
url: https://lwn.net/Articles/1081230/
published: "2026-07-04T16:46:31Z"
feed: lwn
guid: https://lwn.net/Articles/1081230/
---

# Seven stable kernels for Saturday including two security fixes

Greg Kroah-Hartman has announced the release of the [7.1.3](https://lwn.net/Articles/1081231/), [6.18.38](https://lwn.net/Articles/1081232/), [6.12.95](https://lwn.net/Articles/1081233/), [6.6.144](https://lwn.net/Articles/1081234/), [6.1.177](https://lwn.net/Articles/1081235/), [5.15.211](https://lwn.net/Articles/1081236/), and [5.10.260](https://lwn.net/Articles/1081237/) stable kernels. Several kernels in this batch include [a
fix](https://lwn.net/ml/all/2026070450-CVE-2026-53362-fe9b%40gregkh/) for a vulnerability introduced in the 6.0 kernel in IPv6 ( [CVE-2026-53362](https://www.cve.org/CVERecord?id=CVE-2026-53362)), which [could
allow an attacker to escape a container and gain root access](https://access.redhat.com/security/vulnerabilities/RHSB-2026-009).

There is also [a
fix](https://lwn.net/ml/all/2026070403-CVE-2026-53359-4f57%40gregkh/) for a use-after-free bug in KVM ( [CVE-2026-53359](https://www.cve.org/CVERecord?id=CVE-2026-53359)) that was introduced in the 2.6.36 kernel. As usual, each stable kernel includes a number of fixes throughout the tree. Users are advised to upgrade.
