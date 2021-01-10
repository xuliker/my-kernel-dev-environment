

#### [RFC PATCH kvm-unit-tests 0/4] add generic stress test
##### From: Paolo Bonzini <pbonzini@redhat.com>

```c
From patchwork Wed Dec 23 01:08:46 2020
Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
X-Patchwork-Submitter: Paolo Bonzini <pbonzini@redhat.com>
X-Patchwork-Id: 11987425
Return-Path: <kvm-owner@kernel.org>
X-Spam-Checker-Version: SpamAssassin 3.4.0 (2014-02-07) on
	aws-us-west-2-korg-lkml-1.web.codeaurora.org
X-Spam-Level: 
X-Spam-Status: No, score=-11.8 required=3.0 tests=BAYES_00,DKIM_SIGNED,
	DKIM_VALID,HEADER_FROM_DIFFERENT_DOMAINS,INCLUDES_PATCH,MAILING_LIST_MULTI,
	SPF_HELO_NONE,SPF_PASS,USER_AGENT_GIT autolearn=ham autolearn_force=no
	version=3.4.0
Received: from mail.kernel.org (mail.kernel.org [198.145.29.99])
	by smtp.lore.kernel.org (Postfix) with ESMTP id E9495C433DB
	for <kvm@archiver.kernel.org>; Wed, 23 Dec 2020 01:09:50 +0000 (UTC)
Received: from vger.kernel.org (vger.kernel.org [23.128.96.18])
	by mail.kernel.org (Postfix) with ESMTP id B1FC42245C
	for <kvm@archiver.kernel.org>; Wed, 23 Dec 2020 01:09:50 +0000 (UTC)
Received: (majordomo@vger.kernel.org) by vger.kernel.org via listexpand
        id S1725962AbgLWBJe (ORCPT <rfc822;kvm@archiver.kernel.org>);
        Tue, 22 Dec 2020 20:09:34 -0500
Received: from lindbergh.monkeyblade.net ([23.128.96.19]:53346 "EHLO
        lindbergh.monkeyblade.net" rhost-flags-OK-OK-OK-OK) by vger.kernel.org
        with ESMTP id S1725300AbgLWBJe (ORCPT <rfc822;kvm@vger.kernel.org>);
        Tue, 22 Dec 2020 20:09:34 -0500
Received: from mail-wm1-x332.google.com (mail-wm1-x332.google.com
 [IPv6:2a00:1450:4864:20::332])
        by lindbergh.monkeyblade.net (Postfix) with ESMTPS id 8A404C0613D6
        for <kvm@vger.kernel.org>; Tue, 22 Dec 2020 17:08:53 -0800 (PST)
Received: by mail-wm1-x332.google.com with SMTP id y23so4561432wmi.1
        for <kvm@vger.kernel.org>; Tue, 22 Dec 2020 17:08:53 -0800 (PST)
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=gmail.com; s=20161025;
        h=sender:from:to:cc:subject:date:message-id:mime-version
         :content-transfer-encoding;
        bh=ECso56Xc8gg3DWYGpGZLWGbsXUs03FGYbkNr2pEUBUI=;
        b=cdcIKhtLZT9+NKhpn/mN1e9X11O/WJHWl5yNw930VVxF0aFNKo5tlvOV7L2x4iK4sN
         PBV7RyVNakPSeErtz54aWxCeDgnxY8hQi0jgpsSEMjetaMwMYyTfiQQNsD3lnJ88wbsn
         F8hEsvn5wJd3gjRSspkEWrx807wJREwTkehukfybAh1QoL8sXsg81XxN70jwb8ESPsPp
         ZLqW1H/Qr3FK2HiWuNT4ifEeeOBzuw2qU5eydpCqkbstQI0Jqk8XcP4rzuoCIUpwdjfq
         ZMuoRRX2I5zV2GGSoOE8hCE6+Pkrb1C3Ct1fffW1NaqXEeZwiB1lvJ8+eiRcgXJc3+vi
         F9xA==
X-Google-DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=1e100.net; s=20161025;
        h=x-gm-message-state:sender:from:to:cc:subject:date:message-id
         :mime-version:content-transfer-encoding;
        bh=ECso56Xc8gg3DWYGpGZLWGbsXUs03FGYbkNr2pEUBUI=;
        b=HVMQHszjgfCHpp5gKqTX41vtWl3hEsQAWH2D6si+T5FOH19Fgh+1cuxWjr9lrnqpfO
         SXJ6DPRVT8ufnZ59FXaZ9B3G2aQgs6zNwvBgWkPiU+MvbHY8JIYmj8nSub4NBIvNHift
         HkwyGAb7l3ktnapkXpXHy+twoP9KhbC9a/YUyI4e++uIbQkSM1HTzjTjWKIQV/g0g3wV
         4hMghbc8sc09EhTwCXJKh0ula9VsKtRL8i5DbJF3Ysyz14bB0aYbwMBh3fWfLsDlnu2j
         Eo4yU10TT6MYVH2awao3H3c7mtz+Az5ylh0O0vbXjOcY350YShKRz/Q9zT4klyQQDtkQ
         8NUg==
X-Gm-Message-State: AOAM53096NmQsCS3TIko7vAuepvmGIYT3ikran9DFtuyLYXoFmYRAicM
        sJ5nM2jj9J8Rn4lr6h5d9JO4aEgFBEs=
X-Google-Smtp-Source: 
 ABdhPJyca1sK5K7n20dza05betg8clXJGI8tC1d2IzTBKazUx1h0kHUrHCEBayRAt9bytRth+rx7WQ==
X-Received: by 2002:a05:600c:250:: with SMTP id
 16mr8099807wmj.6.1608685732199;
        Tue, 22 Dec 2020 17:08:52 -0800 (PST)
Received: from avogadro.lan ([2001:b07:6468:f312:5e2c:eb9a:a8b6:fd3e])
        by smtp.gmail.com with ESMTPSA id
 h83sm30995047wmf.9.2020.12.22.17.08.51
        (version=TLS1_3 cipher=TLS_AES_256_GCM_SHA384 bits=256/256);
        Tue, 22 Dec 2020 17:08:51 -0800 (PST)
Sender: Paolo Bonzini <paolo.bonzini@gmail.com>
From: Paolo Bonzini <pbonzini@redhat.com>
To: kvm@vger.kernel.org
Cc: mlevitsk@redhat.com
Subject: [RFC PATCH kvm-unit-tests 0/4] add generic stress test
Date: Wed, 23 Dec 2020 02:08:46 +0100
Message-Id: <20201223010850.111882-1-pbonzini@redhat.com>
X-Mailer: git-send-email 2.29.2
MIME-Version: 1.0
Precedence: bulk
List-ID: <kvm.vger.kernel.org>
X-Mailing-List: kvm@vger.kernel.org

This short series adds a generic stress test to KVM unit tests that runs a
series of

The test could grow a lot more features, including:

- wrapping the stress test with a VMX or SVM veneer which would forward
  or inject interrupts periodically

- test perf events

- do some work in the MSI handler, so that they have a chance
  of overlapping

- use PV EOI

- play with TPR and self IPIs, similar to Windows DPCs.

The configuration of the test is set individually for each VCPU on
the command line, for example:

   ./x86/run x86/chaos.flat -smp 2 \
      -append 'invtlb=1,mem=12,hz=100  hz=250,edu=1,edu_hz=53,hlt' -device edu

runs a continuous INVLPG+write test on 1<<12 pages on CPU 0, interrupted
by a 100 Hz timer tick; and keeps CPU 1 mostly idle except for 250 timer
ticks and 53 edu device interrupts per second.

For now, the test runs for an infinite time so it's not included in
unittests.cfg.  Do you think this is worth including in kvm-unit-tests,
and if so are you interested in non-x86 versions of it?  Or should the
code be as pluggable as possible to make it easier to port it?

Thanks,

Paolo

Paolo Bonzini (4):
  libcflat: add a few more runtime functions
  chaos: add generic stress test
  chaos: add timer interrupt to the workload
  chaos: add edu device interrupt to the workload

 lib/alloc.c         |   9 +-
 lib/alloc.h         |   1 +
 lib/libcflat.h      |   4 +-
 lib/string.c        |  59 +++++++++-
 lib/string.h        |   4 +
 lib/x86/processor.h |   2 +-
 x86/Makefile.x86_64 |   1 +
 x86/chaos.c         | 263 ++++++++++++++++++++++++++++++++++++++++++++
 8 files changed, 337 insertions(+), 6 deletions(-)
 create mode 100644 x86/chaos.c
```
#### [PATCH v13 00/15] s390/vfio-ap: dynamic configuration support
##### From: Tony Krowiak <akrowiak@linux.ibm.com>

```c
From patchwork Wed Dec 23 01:15:51 2020
Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
X-Patchwork-Submitter: Tony Krowiak <akrowiak@linux.ibm.com>
X-Patchwork-Id: 11987465
Return-Path: <kvm-owner@kernel.org>
X-Spam-Checker-Version: SpamAssassin 3.4.0 (2014-02-07) on
	aws-us-west-2-korg-lkml-1.web.codeaurora.org
X-Spam-Level: 
X-Spam-Status: No, score=-11.8 required=3.0 tests=BAYES_00,DKIM_SIGNED,
	DKIM_VALID,HEADER_FROM_DIFFERENT_DOMAINS,INCLUDES_PATCH,MAILING_LIST_MULTI,
	SPF_HELO_NONE,SPF_PASS,USER_AGENT_GIT autolearn=unavailable
	autolearn_force=no version=3.4.0
Received: from mail.kernel.org (mail.kernel.org [198.145.29.99])
	by smtp.lore.kernel.org (Postfix) with ESMTP id 811A8C4332B
	for <kvm@archiver.kernel.org>; Wed, 23 Dec 2020 01:18:37 +0000 (UTC)
Received: from vger.kernel.org (vger.kernel.org [23.128.96.18])
	by mail.kernel.org (Postfix) with ESMTP id 56E0E22287
	for <kvm@archiver.kernel.org>; Wed, 23 Dec 2020 01:18:37 +0000 (UTC)
Received: (majordomo@vger.kernel.org) by vger.kernel.org via listexpand
        id S1726779AbgLWBQ7 (ORCPT <rfc822;kvm@archiver.kernel.org>);
        Tue, 22 Dec 2020 20:16:59 -0500
Received: from mx0a-001b2d01.pphosted.com ([148.163.156.1]:36724 "EHLO
        mx0a-001b2d01.pphosted.com" rhost-flags-OK-OK-OK-OK)
        by vger.kernel.org with ESMTP id S1725902AbgLWBQ6 (ORCPT
        <rfc822;kvm@vger.kernel.org>); Tue, 22 Dec 2020 20:16:58 -0500
Received: from pps.filterd (m0098399.ppops.net [127.0.0.1])
        by mx0a-001b2d01.pphosted.com (8.16.0.42/8.16.0.42) with SMTP id
 0BN11pDk192375;
        Tue, 22 Dec 2020 20:16:16 -0500
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=ibm.com;
 h=from : to : cc : subject
 : date : message-id : mime-version : content-transfer-encoding; s=pp1;
 bh=cLfyZ89DOhYu0JoCLBKPgINDEkBUbsOzatsILjZxlAA=;
 b=Tz5lcJ7iUbGBeh1t7duDQ3wrUhRVjSHn5atrKvcalAjhLXoiPD96ovsTSVKpWZAdhk8R
 5V6c20WSPDPmBsatAmiwLeYtECrhVCqqykvlCgEPbxOn40rmKk6lZxOtIHR96lJeHOhG
 nXkcVIim9riZh/8V8P+g8usMU5TfrSHySIK94Xx4iTdNpTTLlcT48bLSKT979w3pcHZI
 IGJLh1/rVqGoP+h+FUJF8NnSjUUzAnkJtsbht9h6VZxENm/vqdvRO3+Rzc6S+EqLmUkX
 bHrDDh0CC+Jr6UuWeE0z4sXp1F5tw7NiINdqzcb/SZMGCampc+qaz+SkPFqtrXDeIiw4 BQ==
Received: from pps.reinject (localhost [127.0.0.1])
        by mx0a-001b2d01.pphosted.com with ESMTP id 35ktw79e2d-1
        (version=TLSv1.2 cipher=ECDHE-RSA-AES256-GCM-SHA384 bits=256
 verify=NOT);
        Tue, 22 Dec 2020 20:16:16 -0500
Received: from m0098399.ppops.net (m0098399.ppops.net [127.0.0.1])
        by pps.reinject (8.16.0.36/8.16.0.36) with SMTP id 0BN12Bjv193933;
        Tue, 22 Dec 2020 20:16:16 -0500
Received: from ppma02dal.us.ibm.com (a.bd.3ea9.ip4.static.sl-reverse.com
 [169.62.189.10])
        by mx0a-001b2d01.pphosted.com with ESMTP id 35ktw79e27-1
        (version=TLSv1.2 cipher=ECDHE-RSA-AES256-GCM-SHA384 bits=256
 verify=NOT);
        Tue, 22 Dec 2020 20:16:15 -0500
Received: from pps.filterd (ppma02dal.us.ibm.com [127.0.0.1])
        by ppma02dal.us.ibm.com (8.16.0.42/8.16.0.42) with SMTP id
 0BN1DEoI004574;
        Wed, 23 Dec 2020 01:16:15 GMT
Received: from b01cxnp22036.gho.pok.ibm.com (b01cxnp22036.gho.pok.ibm.com
 [9.57.198.26])
        by ppma02dal.us.ibm.com with ESMTP id 35kj7qvbpd-1
        (version=TLSv1.2 cipher=ECDHE-RSA-AES256-GCM-SHA384 bits=256
 verify=NOT);
        Wed, 23 Dec 2020 01:16:15 +0000
Received: from b01ledav004.gho.pok.ibm.com (b01ledav004.gho.pok.ibm.com
 [9.57.199.109])
        by b01cxnp22036.gho.pok.ibm.com (8.14.9/8.14.9/NCO v10.0) with ESMTP
 id 0BN1GD7B8585980
        (version=TLSv1/SSLv3 cipher=DHE-RSA-AES256-GCM-SHA384 bits=256
 verify=OK);
        Wed, 23 Dec 2020 01:16:13 GMT
Received: from b01ledav004.gho.pok.ibm.com (unknown [127.0.0.1])
        by IMSVA (Postfix) with ESMTP id 439E9112063;
        Wed, 23 Dec 2020 01:16:13 +0000 (GMT)
Received: from b01ledav004.gho.pok.ibm.com (unknown [127.0.0.1])
        by IMSVA (Postfix) with ESMTP id 6636D112065;
        Wed, 23 Dec 2020 01:16:12 +0000 (GMT)
Received: from cpe-66-24-58-13.stny.res.rr.com.com (unknown [9.85.193.150])
        by b01ledav004.gho.pok.ibm.com (Postfix) with ESMTP;
        Wed, 23 Dec 2020 01:16:12 +0000 (GMT)
From: Tony Krowiak <akrowiak@linux.ibm.com>
To: linux-s390@vger.kernel.org, linux-kernel@vger.kernel.org,
        kvm@vger.kernel.org
Cc: freude@linux.ibm.com, borntraeger@de.ibm.com, cohuck@redhat.com,
        mjrosato@linux.ibm.com, pasic@linux.ibm.com,
        alex.williamson@redhat.com, kwankhede@nvidia.com,
        fiuczy@linux.ibm.com, frankja@linux.ibm.com, david@redhat.com,
        hca@linux.ibm.com, gor@linux.ibm.com,
        Tony Krowiak <akrowiak@linux.ibm.com>
Subject: [PATCH v13 00/15] s390/vfio-ap: dynamic configuration support
Date: Tue, 22 Dec 2020 20:15:51 -0500
Message-Id: <20201223011606.5265-1-akrowiak@linux.ibm.com>
X-Mailer: git-send-email 2.21.1
MIME-Version: 1.0
X-TM-AS-GCONF: 00
X-Proofpoint-Virus-Version: vendor=fsecure engine=2.50.10434:6.0.343,18.0.737
 definitions=2020-12-22_13:2020-12-21,2020-12-22 signatures=0
X-Proofpoint-Spam-Details: rule=outbound_notspam policy=outbound score=0
 suspectscore=0 malwarescore=0
 spamscore=0 lowpriorityscore=0 adultscore=0 priorityscore=1501
 impostorscore=0 bulkscore=0 mlxlogscore=999 phishscore=0 clxscore=1015
 mlxscore=0 classifier=spam adjust=0 reason=mlx scancount=1
 engine=8.12.0-2009150000 definitions=main-2012230003
Precedence: bulk
List-ID: <kvm.vger.kernel.org>
X-Mailing-List: kvm@vger.kernel.org

Note: Patch 1, s390/vfio-ap: clean up vfio_ap resources when KVM
      pointer invalidated does not belong to this series. It has been
      posted as a separate patch to fix a known problem. It is included
      here because it will likely pre-req for this series.

The current design for AP pass-through does not support making dynamic
changes to the AP matrix of a running guest resulting in a few 
deficiencies this patch series is intended to mitigate:

1. Adapters, domains and control domains can not be added to or removed
   from a running guest. In order to modify a guest's AP configuration,
   the guest must be terminated; only then can AP resources be assigned
   to or unassigned from the guest's matrix mdev. The new AP 
   configuration becomes available to the guest when it is subsequently
   restarted.

2. The AP bus's /sys/bus/ap/apmask and /sys/bus/ap/aqmask interfaces can
   be modified by a root user without any restrictions. A change to
   either mask can result in AP queue devices being unbound from the
   vfio_ap device driver and bound to a zcrypt device driver even if a
   guest is using the queues, thus giving the host access to the guest's
   private crypto data and vice versa.

3. The APQNs derived from the Cartesian product of the APIDs of the
   adapters and APQIs of the domains assigned to a matrix mdev must
   reference an AP queue device bound to the vfio_ap device driver. The
   AP architecture allows assignment of AP resources that are not
   available to the system, so this artificial restriction is not 
   compliant with the architecture.

4. The AP configuration profile can be dynamically changed for the linux
   host after a KVM guest is started. For example, a new domain can be
   dynamically added to the configuration profile via the SE or an HMC
   connected to a DPM enabled lpar. Likewise, AP adapters can be 
   dynamically configured (online state) and deconfigured (standby state)
   using the SE, an SCLP command or an HMC connected to a DPM enabled
   lpar. This can result in inadvertent sharing of AP queues between the
   guest and host.

5. A root user can manually unbind an AP queue device representing a 
   queue in use by a KVM guest via the vfio_ap device driver's sysfs 
   unbind attribute. In this case, the guest will be using a queue that
   is not bound to the driver which violates the device model.

This patch series introduces the following changes to the current design
to alleviate the shortcomings described above as well as to implement
more of the AP architecture:

1. A root user will be prevented from making edits to the AP bus's
   /sys/bus/ap/apmask or /sys/bus/ap/aqmask if the change would transfer
   ownership of an APQN from the vfio_ap device driver to a zcrypt driver
   while the APQN is assigned to a matrix mdev.

2. Allow a root user to hot plug/unplug AP adapters, domains and control
   domains for a KVM guest using the matrix mdev via its sysfs
   assign/unassign attributes.

4. Allow assignment of an AP adapter or domain to a matrix mdev even if
   it results in assignment of an APQN that does not reference an AP
   queue device bound to the vfio_ap device driver, as long as the APQN
   is not reserved for use by the default zcrypt drivers (also known as
   over-provisioning of AP resources). Allowing over-provisioning of AP
   resources better models the architecture which does not preclude
   assigning AP resources that are not yet available in the system. Such
   APQNs, however, will not be assigned to the guest using the matrix
   mdev; only APQNs referencing AP queue devices bound to the vfio_ap
   device driver will actually get assigned to the guest.

5. Handle dynamic changes to the AP device model. 

1. Rationale for changes to AP bus's apmask/aqmask interfaces:
----------------------------------------------------------
Due to the extremely sensitive nature of cryptographic data, it is
imperative that great care be taken to ensure that such data is secured.
Allowing a root user, either inadvertently or maliciously, to configure
these masks such that a queue is shared between the host and a guest is
not only avoidable, it is advisable. It was suggested that this scenario
is better handled in user space with management software, but that does
not preclude a malicious administrator from using the sysfs interfaces
to gain access to a guest's crypto data. It was also suggested that this
scenario could be avoided by taking access to the adapter away from the
guest and zeroing out the queues prior to the vfio_ap driver releasing the
device; however, stealing an adapter in use from a guest as a by-product
of an operation is bad and will likely cause problems for the guest
unnecessarily. It was decided that the most effective solution with the
least number of negative side effects is to prevent the situation at the
source.

2. Rationale for hot plug/unplug using matrix mdev sysfs interfaces:
----------------------------------------------------------------
Allowing a user to hot plug/unplug AP resources using the matrix mdev
sysfs interfaces circumvents the need to terminate the guest in order to
modify its AP configuration. Allowing dynamic configuration makes 
reconfiguring a guest's AP matrix much less disruptive.

3. Rationale for allowing over-provisioning of AP resources:
----------------------------------------------------------- 
Allowing assignment of AP resources to a matrix mdev and ultimately to a
guest better models the AP architecture. The architecture does not
preclude assignment of unavailable AP resources. If a queue subsequently
becomes available while a guest using the matrix mdev to which its APQN
is assigned, the guest will be given access to it. If an APQN
is dynamically unassigned from the underlying host system, it will 
automatically become unavailable to the guest.

Change log v12-v13:
------------------
* Combined patches 12/13 from previous series into one patch

* Moved all changes for linking queues and mdevs into a single patch

* Re-ordered some patches to aid in review

* Using mutex_trylock() function in adapter/domain assignment functions
  to avoid potential deadlock condition with in_use callback

* Using filtering function for refreshing the guest's APCB for all events
  that change the APCB: assign/unassign adapters, domains, control domains;
  bind/unbind of queue devices; and, changes to the host AP configuration.

Change log v11-v12:
------------------
* Moved matrix device lock to protect group notifier callback

* Split the 'No need to disable IRQ after queue reset' patch into
  multiple patches for easier review (move probe/remove callback
  functions and remove disable IRQ after queue reset)

* Added code to decrement reference count for KVM in group notifier
  callback

* Using mutex_trylock() in functions implementing the sysfs assign_adapter
  and assign_domain as well as the in_use callback to avoid deadlock 
  between the AP bus's ap_perms mutex and the matrix device lock used by
  vfio_ap driver.

* The sysfs guest_matrix attribute of the vfio_ap mdev will now display
  the shadow APCB regardless of whether a guest is using the mdev or not

* Replaced vfio_ap mdev filtering function with a function that initializes
  the guest's APCB by filtering the vfio_ap mdev by APID.

* No longer using filtering function during adapter/domain assignment
  to/from the vfio_ap mdev; replaced with new hot plug/unplug 
  adapter/domain functions.

* No longer using filtering function during bind/unbind; replaced with
  hot plug/unplug queue functions.

* No longer using filtering function for bulk assignment of new adapters
  and domains in on_scan_complete callback; replaced with new hot plug
  functions.    
  

Change log v10-v11:
------------------
* The matrix mdev's configuration is not filtered by APID so that if any
  APQN assigned to the mdev is not bound to the vfio_ap device driver,
  the adapter will not get plugged into the KVM guest on startup, or when
  a new adapter is assigned to the mdev.

* Replaced patch 8 by squashing patches 8 (filtering patch) and 15 (handle 
  probe/remove).

* Added a patch 1 to remove disable IRQ after a reset because the reset
  already disables a queue.

* Now using filtering code to update the KVM guest's matrix when
  notified that AP bus scan has completed.

* Fixed issue with probe/remove not inititiated by a configuration change
  occurring within a config change.


Change log v9-v10:
-----------------
* Updated the documentation in vfio-ap.rst to include information about the
  AP dynamic configuration support

Change log v8-v9:
----------------
* Fixed errors flagged by the kernel test robot

* Fixed issue with guest losing queues when a new queue is probed due to
  manual bind operation.

Change log v7-v8:
----------------
* Now logging a message when an attempt to reserve APQNs for the zcrypt
  drivers will result in taking a queue away from a KVM guest to provide
  the sysadmin a way to ascertain why the sysfs operation failed.

* Created locked and unlocked versions of the ap_parse_mask_str() function.

* Now using new interface provided by an AP bus patch -
  s390/ap: introduce new ap function ap_get_qdev() - to retrieve
  struct ap_queue representing an AP queue device. This patch is not a
  part of this series but is a prerequisite for this series. 

Change log v6-v7:
----------------
* Added callbacks to AP bus:
  - on_config_changed: Notifies implementing drivers that
    the AP configuration has changed since last AP device scan.
  - on_scan_complete: Notifies implementing drivers that the device scan
    has completed.
  - implemented on_config_changed and on_scan_complete callbacks for
    vfio_ap device driver.
  - updated vfio_ap device driver's probe and remove callbacks to handle
    dynamic changes to the AP device model. 
* Added code to filter APQNs when assigning AP resources to a KVM guest's
  CRYCB

Change log v5-v6:
----------------
* Fixed a bug in ap_bus.c introduced with patch 2/7 of the v5 
  series. Harald Freudenberer pointed out that the mutex lock
  for ap_perms_mutex in the apmask_store and aqmask_store functions
  was not being freed. 

* Removed patch 6/7 which added logging to the vfio_ap driver
  to expedite acceptance of this series. The logging will be introduced
  with a separate patch series to allow more time to explore options
  such as DBF logging vs. tracepoints.

* Added 3 patches related to ensuring that APQNs that do not reference
  AP queue devices bound to the vfio_ap device driver are not assigned
  to the guest CRYCB:

  Patch 4: Filter CRYCB bits for unavailable queue devices
  Patch 5: sysfs attribute to display the guest CRYCB
  Patch 6: update guest CRYCB in vfio_ap probe and remove callbacks

* Added a patch (Patch 9) to version the vfio_ap module.

* Reshuffled patches to allow the in_use callback implementation to
  invoke the vfio_ap_mdev_verify_no_sharing() function introduced in
  patch 2. 

Change log v4-v5:
----------------
* Added a patch to provide kernel s390dbf debug logs for VFIO AP

Change log v3->v4:
-----------------
* Restored patches preventing root user from changing ownership of
  APQNs from zcrypt drivers to the vfio_ap driver if the APQN is
  assigned to an mdev.

* No longer enforcing requirement restricting guest access to
  queues represented by a queue device bound to the vfio_ap
  device driver.

* Removed shadow CRYCB and now directly updating the guest CRYCB
  from the matrix mdev's matrix.

* Rebased the patch series on top of 'vfio: ap: AP Queue Interrupt
  Control' patches.

* Disabled bind/unbind sysfs interfaces for vfio_ap driver

Change log v2->v3:
-----------------
* Allow guest access to an AP queue only if the queue is bound to
  the vfio_ap device driver.

* Removed the patch to test CRYCB masks before taking the vCPUs
  out of SIE. Now checking the shadow CRYCB in the vfio_ap driver.

Change log v1->v2:
-----------------
* Removed patches preventing root user from unbinding AP queues from 
  the vfio_ap device driver
* Introduced a shadow CRYCB in the vfio_ap driver to manage dynamic 
  changes to the AP guest configuration due to root user interventions
  or hardware anomalies.

Tony Krowiak (15):
  s390/vfio-ap: clean up vfio_ap resources when KVM pointer invalidated
  s390/vfio-ap: No need to disable IRQ after queue reset
  s390/vfio-ap: move probe and remove callbacks to vfio_ap_ops.c
  s390/vfio-ap: use new AP bus interface to search for queue devices
  s390/vfio-ap: manage link between queue struct and matrix mdev
  s390/vfio-ap: allow assignment of unavailable AP queues to mdev device
  s390/vfio-ap: introduce shadow APCB
  s390/vfio-ap: sysfs attribute to display the guest's matrix
  s390/vfio-ap: allow hot plug/unplug of AP resources using mdev device
  s390/zcrypt: driver callback to indicate resource in use
  s390/vfio-ap: implement in-use callback for vfio_ap driver
  s390/zcrypt: Notify driver on config changed and scan complete
    callbacks
  s390/vfio-ap: handle host AP config change notification
  s390/vfio-ap: handle AP bus scan completed notification
  s390/vfio-ap: update docs to include dynamic config support

 Documentation/s390/vfio-ap.rst        | 383 ++++++++---
 drivers/s390/crypto/ap_bus.c          | 251 +++++++-
 drivers/s390/crypto/ap_bus.h          |  16 +
 drivers/s390/crypto/vfio_ap_drv.c     |  50 +-
 drivers/s390/crypto/vfio_ap_ops.c     | 891 +++++++++++++++++---------
 drivers/s390/crypto/vfio_ap_private.h |  29 +-
 6 files changed, 1170 insertions(+), 450 deletions(-)
```