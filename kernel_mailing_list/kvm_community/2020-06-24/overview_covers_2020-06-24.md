

#### [PATCH v3 00/14] vfio: expose virtual Shared Virtual Addressing to
##### From: Liu Yi L <yi.l.liu@intel.com>

```c
From patchwork Wed Jun 24 08:55:13 2020
Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
X-Patchwork-Submitter: Liu Yi L <yi.l.liu@intel.com>
X-Patchwork-Id: 11622685
Return-Path: <SRS0=aKHR=AF=vger.kernel.org=kvm-owner@kernel.org>
Received: from mail.kernel.org (pdx-korg-mail-1.web.codeaurora.org
 [172.30.200.123])
	by pdx-korg-patchwork-2.web.codeaurora.org (Postfix) with ESMTP id 4B877912
	for <patchwork-kvm@patchwork.kernel.org>;
 Wed, 24 Jun 2020 08:50:51 +0000 (UTC)
Received: from vger.kernel.org (vger.kernel.org [23.128.96.18])
	by mail.kernel.org (Postfix) with ESMTP id 3519E20C09
	for <patchwork-kvm@patchwork.kernel.org>;
 Wed, 24 Jun 2020 08:50:51 +0000 (UTC)
Received: (majordomo@vger.kernel.org) by vger.kernel.org via listexpand
        id S2389600AbgFXIup (ORCPT
        <rfc822;patchwork-kvm@patchwork.kernel.org>);
        Wed, 24 Jun 2020 04:50:45 -0400
Received: from mga01.intel.com ([192.55.52.88]:1309 "EHLO mga01.intel.com"
        rhost-flags-OK-OK-OK-OK) by vger.kernel.org with ESMTP
        id S2387796AbgFXIs6 (ORCPT <rfc822;kvm@vger.kernel.org>);
        Wed, 24 Jun 2020 04:48:58 -0400
IronPort-SDR: 
 dof78PE56RueGZuaA/IkzjRmMN5qmqtInXwqzspmx5gul6ERErijfWmgRrfUMqwX9GlyQDsbTO
 yEgEt9AqXhxg==
X-IronPort-AV: E=McAfee;i="6000,8403,9661"; a="162484865"
X-IronPort-AV: E=Sophos;i="5.75,274,1589266800";
   d="scan'208";a="162484865"
X-Amp-Result: SKIPPED(no attachment in message)
X-Amp-File-Uploaded: False
Received: from orsmga003.jf.intel.com ([10.7.209.27])
  by fmsmga101.fm.intel.com with ESMTP/TLS/ECDHE-RSA-AES256-GCM-SHA384;
 24 Jun 2020 01:48:56 -0700
IronPort-SDR: 
 3wjsszL64qt/U6/qcUJlWSGelCbu9HW5E0/evm5z894kj3Dtspdpx+1+N9J8/5oAYE3DaDplnU
 Sg64TNcC6g2g==
X-ExtLoop1: 1
X-IronPort-AV: E=Sophos;i="5.75,274,1589266800";
   d="scan'208";a="275624490"
Received: from jacob-builder.jf.intel.com ([10.7.199.155])
  by orsmga003.jf.intel.com with ESMTP; 24 Jun 2020 01:48:55 -0700
From: Liu Yi L <yi.l.liu@intel.com>
To: alex.williamson@redhat.com, eric.auger@redhat.com,
        baolu.lu@linux.intel.com, joro@8bytes.org
Cc: kevin.tian@intel.com, jacob.jun.pan@linux.intel.com,
        ashok.raj@intel.com, yi.l.liu@intel.com, jun.j.tian@intel.com,
        yi.y.sun@intel.com, jean-philippe@linaro.org, peterx@redhat.com,
        hao.wu@intel.com, iommu@lists.linux-foundation.org,
        kvm@vger.kernel.org, linux-kernel@vger.kernel.org
Subject: [PATCH v3 00/14] vfio: expose virtual Shared Virtual Addressing to
 VMs
Date: Wed, 24 Jun 2020 01:55:13 -0700
Message-Id: <1592988927-48009-1-git-send-email-yi.l.liu@intel.com>
X-Mailer: git-send-email 2.7.4
Sender: kvm-owner@vger.kernel.org
Precedence: bulk
List-ID: <kvm.vger.kernel.org>
X-Mailing-List: kvm@vger.kernel.org

Shared Virtual Addressing (SVA), a.k.a, Shared Virtual Memory (SVM) on
Intel platforms allows address space sharing between device DMA and
applications. SVA can reduce programming complexity and enhance security.

This VFIO series is intended to expose SVA usage to VMs. i.e. Sharing
guest application address space with passthru devices. This is called
vSVA in this series. The whole vSVA enabling requires QEMU/VFIO/IOMMU
changes. For IOMMU and QEMU changes, they are in separate series (listed
in the "Related series").

The high-level architecture for SVA virtualization is as below, the key
design of vSVA support is to utilize the dual-stage IOMMU translation (
also known as IOMMU nesting translation) capability in host IOMMU.


    .-------------.  .---------------------------.
    |   vIOMMU    |  | Guest process CR3, FL only|
    |             |  '---------------------------'
    .----------------/
    | PASID Entry |--- PASID cache flush -
    '-------------'                       |
    |             |                       V
    |             |                CR3 in GPA
    '-------------'
Guest
------| Shadow |--------------------------|--------
      v        v                          v
Host
    .-------------.  .----------------------.
    |   pIOMMU    |  | Bind FL for GVA-GPA  |
    |             |  '----------------------'
    .----------------/  |
    | PASID Entry |     V (Nested xlate)
    '----------------\.------------------------------.
    |             |   |SL for GPA-HPA, default domain|
    |             |   '------------------------------'
    '-------------'
Where:
 - FL = First level/stage one page tables
 - SL = Second level/stage two page tables

Patch Overview:
 1. a refactor to vfio_iommu_type1 ioctl (patch 0001)
 2. reports IOMMU nesting info to userspace ( patch 0002, 0003 and 0014)
 3. vfio support for PASID allocation and free for VMs (patch 0004, 0005, 0006)
 4. vfio support for binding guest page table to host (patch 0007, 0008, 0009)
 5. vfio support for IOMMU cache invalidation from VMs (patch 0010)
 6. vfio support for vSVA usage on IOMMU-backed mdevs (patch 0011)
 7. expose PASID capability to VM (patch 0012)
 8. add doc for VFIO dual stage control (patch 0013)

The complete vSVA kernel upstream patches are divided into three phases:
    1. Common APIs and PCI device direct assignment
    2. IOMMU-backed Mediated Device assignment
    3. Page Request Services (PRS) support

This patchset is aiming for the phase 1 and phase 2, and based on Jacob's
below series.
[PATCH v3 0/5] IOMMU user API enhancement - wip
https://lore.kernel.org/linux-iommu/1592931837-58223-1-git-send-email-jacob.jun.pan@linux.intel.com/

[PATCH 00/10] IOASID extensions for guest SVA - wip
https://lkml.org/lkml/2020/3/25/874

The latest IOASID code added below new interface for itertate all PASIDs of an
ioasid_set. The implementation is not sent out yet as Jacob needs some cleanup,
it can be found in branch vsva-linux-5.8-rc1-v3
 int ioasid_set_for_each_ioasid(int sid, void (*fn)(ioasid_t id, void *data), void *data);

Complete set for current vSVA can be found in below branch.
This branch also includes some extra modifications to IOASID core code and
vt-d iommu driver cleanup patches.
https://github.com/luxis1999/linux-vsva.git:vsva-linux-5.8-rc1-v3

The corresponding QEMU patch series is included in below branch:
https://github.com/luxis1999/qemu.git:vsva_5.8_rc1_qemu_rfcv6


Regards,
Yi Liu

Changelog:
	- Patch v2 -> Patch v3:
	  a) Rebase on top of Jacob's v3 iommu uapi patchset
	  b) Address comments from Kevin and Stefan Hajnoczi
	  c) Reuse DOMAIN_ATTR_NESTING to get iommu nesting info
	  d) Drop [PATCH v2 07/15] iommu/uapi: Add iommu_gpasid_unbind_data
	  https://lore.kernel.org/linux-iommu/1591877734-66527-1-git-send-email-yi.l.liu@intel.com/#r

	- Patch v1 -> Patch v2:
	  a) Refactor vfio_iommu_type1_ioctl() per suggestion from Christoph
	     Hellwig.
	  b) Re-sequence the patch series for better bisect support.
	  c) Report IOMMU nesting cap info in detail instead of a format in
	     v1.
	  d) Enforce one group per nesting type container for vfio iommu type1
	     driver.
	  e) Build the vfio_mm related code from vfio.c to be a separate
	     vfio_pasid.ko.
	  f) Add PASID ownership check in IOMMU driver.
	  g) Adopted to latest IOMMU UAPI design. Removed IOMMU UAPI version
	     check. Added iommu_gpasid_unbind_data for unbind requests from
	     userspace.
	  h) Define a single ioctl:VFIO_IOMMU_NESTING_OP for bind/unbind_gtbl
	     and cahce_invld.
	  i) Document dual stage control in vfio.rst.
	  Patch v1: https://lore.kernel.org/linux-iommu/1584880325-10561-1-git-send-email-yi.l.liu@intel.com/

	- RFC v3 -> Patch v1:
	  a) Address comments to the PASID request(alloc/free) path
	  b) Report PASID alloc/free availabitiy to user-space
	  c) Add a vfio_iommu_type1 parameter to support pasid quota tuning
	  d) Adjusted to latest ioasid code implementation. e.g. remove the
	     code for tracking the allocated PASIDs as latest ioasid code
	     will track it, VFIO could use ioasid_free_set() to free all
	     PASIDs.
	  RFC v3: https://lore.kernel.org/linux-iommu/1580299912-86084-1-git-send-email-yi.l.liu@intel.com/

	- RFC v2 -> v3:
	  a) Refine the whole patchset to fit the roughly parts in this series
	  b) Adds complete vfio PASID management framework. e.g. pasid alloc,
	  free, reclaim in VM crash/down and per-VM PASID quota to prevent
	  PASID abuse.
	  c) Adds IOMMU uAPI version check and page table format check to ensure
	  version compatibility and hardware compatibility.
	  d) Adds vSVA vfio support for IOMMU-backed mdevs.
	  RFC v2: https://lore.kernel.org/linux-iommu/1571919983-3231-1-git-send-email-yi.l.liu@intel.com/

	- RFC v1 -> v2:
	  Dropped vfio: VFIO_IOMMU_ATTACH/DETACH_PASID_TABLE.
	  RFC v1: https://lore.kernel.org/linux-iommu/1562324772-3084-1-git-send-email-yi.l.liu@intel.com/
---
Eric Auger (1):
  vfio: Document dual stage control

Liu Yi L (12):
  vfio/type1: Refactor vfio_iommu_type1_ioctl()
  iommu: Report domain nesting info
  vfio/type1: Report iommu nesting info to userspace
  vfio: Add PASID allocation/free support
  iommu/vt-d: Support setting ioasid set to domain
  vfio/type1: Add VFIO_IOMMU_PASID_REQUEST (alloc/free)
  iommu/vt-d: Check ownership for PASIDs from user-space
  vfio/type1: Support binding guest page tables to PASID
  vfio/type1: Allow invalidating first-level/stage IOMMU cache
  vfio/type1: Add vSVA support for IOMMU-backed mdevs
  vfio/pci: Expose PCIe PASID capability to guest
  iommu/vt-d: Support reporting nesting capability info

Yi Sun (1):
  iommu: Pass domain to sva_unbind_gpasid()

 Documentation/driver-api/vfio.rst  |  67 ++++
 drivers/iommu/arm-smmu-v3.c        |  29 +-
 drivers/iommu/arm-smmu.c           |  29 +-
 drivers/iommu/intel/iommu.c        | 105 ++++-
 drivers/iommu/intel/svm.c          |  10 +-
 drivers/iommu/iommu.c              |   2 +-
 drivers/vfio/Kconfig               |   6 +
 drivers/vfio/Makefile              |   1 +
 drivers/vfio/pci/vfio_pci_config.c |   2 +-
 drivers/vfio/vfio_iommu_type1.c    | 800 +++++++++++++++++++++++++++++--------
 drivers/vfio/vfio_pasid.c          | 191 +++++++++
 include/linux/intel-iommu.h        |  23 +-
 include/linux/iommu.h              |   4 +-
 include/linux/vfio.h               |  54 +++
 include/uapi/linux/iommu.h         |  59 +++
 include/uapi/linux/vfio.h          |  78 ++++
 16 files changed, 1273 insertions(+), 187 deletions(-)
 create mode 100644 drivers/vfio/vfio_pasid.c
```
#### [PATCH 0/2] s390: Add API Docs for DIAGNOSE 0x318 and fix rst
##### From: Collin Walling <walling@linux.ibm.com>

```c
From patchwork Wed Jun 24 20:21:58 2020
Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
X-Patchwork-Submitter: Collin Walling <walling@linux.ibm.com>
X-Patchwork-Id: 11624135
Return-Path: <SRS0=aKHR=AF=vger.kernel.org=kvm-owner@kernel.org>
Received: from mail.kernel.org (pdx-korg-mail-1.web.codeaurora.org
 [172.30.200.123])
	by pdx-korg-patchwork-2.web.codeaurora.org (Postfix) with ESMTP id 03C3C912
	for <patchwork-kvm@patchwork.kernel.org>;
 Wed, 24 Jun 2020 20:22:41 +0000 (UTC)
Received: from vger.kernel.org (vger.kernel.org [23.128.96.18])
	by mail.kernel.org (Postfix) with ESMTP id E36A52080C
	for <patchwork-kvm@patchwork.kernel.org>;
 Wed, 24 Jun 2020 20:22:40 +0000 (UTC)
Received: (majordomo@vger.kernel.org) by vger.kernel.org via listexpand
        id S2391434AbgFXUWj (ORCPT
        <rfc822;patchwork-kvm@patchwork.kernel.org>);
        Wed, 24 Jun 2020 16:22:39 -0400
Received: from mx0a-001b2d01.pphosted.com ([148.163.156.1]:5758 "EHLO
        mx0a-001b2d01.pphosted.com" rhost-flags-OK-OK-OK-OK)
        by vger.kernel.org with ESMTP id S2391054AbgFXUWj (ORCPT
        <rfc822;kvm@vger.kernel.org>); Wed, 24 Jun 2020 16:22:39 -0400
Received: from pps.filterd (m0098393.ppops.net [127.0.0.1])
        by mx0a-001b2d01.pphosted.com (8.16.0.42/8.16.0.42) with SMTP id
 05OK3T45066229;
        Wed, 24 Jun 2020 16:22:38 -0400
Received: from pps.reinject (localhost [127.0.0.1])
        by mx0a-001b2d01.pphosted.com with ESMTP id 31uwyyxjm0-1
        (version=TLSv1.2 cipher=ECDHE-RSA-AES256-GCM-SHA384 bits=256
 verify=NOT);
        Wed, 24 Jun 2020 16:22:38 -0400
Received: from m0098393.ppops.net (m0098393.ppops.net [127.0.0.1])
        by pps.reinject (8.16.0.36/8.16.0.36) with SMTP id 05OK4eEL069697;
        Wed, 24 Jun 2020 16:22:38 -0400
Received: from ppma02wdc.us.ibm.com (aa.5b.37a9.ip4.static.sl-reverse.com
 [169.55.91.170])
        by mx0a-001b2d01.pphosted.com with ESMTP id 31uwyyxjkj-1
        (version=TLSv1.2 cipher=ECDHE-RSA-AES256-GCM-SHA384 bits=256
 verify=NOT);
        Wed, 24 Jun 2020 16:22:38 -0400
Received: from pps.filterd (ppma02wdc.us.ibm.com [127.0.0.1])
        by ppma02wdc.us.ibm.com (8.16.0.42/8.16.0.42) with SMTP id
 05OKKPLI017834;
        Wed, 24 Jun 2020 20:22:37 GMT
Received: from b01cxnp22036.gho.pok.ibm.com (b01cxnp22036.gho.pok.ibm.com
 [9.57.198.26])
        by ppma02wdc.us.ibm.com with ESMTP id 31uus3ph66-1
        (version=TLSv1.2 cipher=ECDHE-RSA-AES256-GCM-SHA384 bits=256
 verify=NOT);
        Wed, 24 Jun 2020 20:22:37 +0000
Received: from b01ledav003.gho.pok.ibm.com (b01ledav003.gho.pok.ibm.com
 [9.57.199.108])
        by b01cxnp22036.gho.pok.ibm.com (8.14.9/8.14.9/NCO v10.0) with ESMTP
 id 05OKMZ2315401788
        (version=TLSv1/SSLv3 cipher=DHE-RSA-AES256-GCM-SHA384 bits=256
 verify=OK);
        Wed, 24 Jun 2020 20:22:35 GMT
Received: from b01ledav003.gho.pok.ibm.com (unknown [127.0.0.1])
        by IMSVA (Postfix) with ESMTP id 298CFB2066;
        Wed, 24 Jun 2020 20:22:35 +0000 (GMT)
Received: from b01ledav003.gho.pok.ibm.com (unknown [127.0.0.1])
        by IMSVA (Postfix) with ESMTP id D1B45B2065;
        Wed, 24 Jun 2020 20:22:34 +0000 (GMT)
Received: from localhost.localdomain.com (unknown [9.85.198.108])
        by b01ledav003.gho.pok.ibm.com (Postfix) with ESMTP;
        Wed, 24 Jun 2020 20:22:34 +0000 (GMT)
From: Collin Walling <walling@linux.ibm.com>
To: kvm@vger.kernel.org, linux-s390@vger.kernel.org
Cc: pbonzini@redhat.com, borntraeger@de.ibm.com, frankja@linux.ibm.com,
        david@redhat.com, cohuck@redhat.com, imbrenda@linux.ibm.com,
        heiko.carstens@de.ibm.com, gor@linux.ibm.com, thuth@redhat.com
Subject: [PATCH 0/2] s390: Add API Docs for DIAGNOSE 0x318 and fix rst
Date: Wed, 24 Jun 2020 16:21:58 -0400
Message-Id: <20200624202200.28209-1-walling@linux.ibm.com>
X-Mailer: git-send-email 2.26.2
MIME-Version: 1.0
X-TM-AS-GCONF: 00
X-Proofpoint-Virus-Version: vendor=fsecure engine=2.50.10434:6.0.216,18.0.687
 definitions=2020-06-24_16:2020-06-24,2020-06-24 signatures=0
X-Proofpoint-Spam-Details: rule=outbound_notspam policy=outbound score=0
 impostorscore=0 spamscore=0
 clxscore=1015 bulkscore=0 priorityscore=1501 cotscore=-2147483648
 mlxscore=0 lowpriorityscore=0 suspectscore=0 adultscore=0 mlxlogscore=803
 phishscore=0 malwarescore=0 classifier=spam adjust=0 reason=mlx
 scancount=1 engine=8.12.0-2004280000 definitions=main-2006240129
Sender: kvm-owner@vger.kernel.org
Precedence: bulk
List-ID: <kvm.vger.kernel.org>
X-Mailing-List: kvm@vger.kernel.org

Adds documentation for the s390-specfic DIAGNOSE 0x318 instruction, as
well as fixes some missing rst symbols for the neighboring entries.

Suggested-by: Cornelia Huck <cohuck@redhat.com>

Collin Walling (2):
  docs: kvm: add documentation for KVM_CAP_S390_DIAG318
  docs: kvm: fix rst formatting

 Documentation/virt/kvm/api.rst | 26 +++++++++++++++++++++++---
 1 file changed, 23 insertions(+), 3 deletions(-)
```
