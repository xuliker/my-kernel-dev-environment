#### [PATCH] KVM: use struct_size() and flex_array_size() helpers in kvm_io_bus_unregister_dev()
##### From: Rustam Kovhaev <rkovhaev@gmail.com>

```c
Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
X-Patchwork-Submitter: Rustam Kovhaev <rkovhaev@gmail.com>
X-Patchwork-Id: 11784757
Return-Path: <SRS0=sqq+=C3=vger.kernel.org=kvm-owner@kernel.org>
Received: from mail.kernel.org (pdx-korg-mail-1.web.codeaurora.org
 [172.30.200.123])
	by pdx-korg-patchwork-2.web.codeaurora.org (Postfix) with ESMTP id 36A456CA
	for <patchwork-kvm@patchwork.kernel.org>;
 Fri, 18 Sep 2020 12:05:15 +0000 (UTC)
Received: from vger.kernel.org (vger.kernel.org [23.128.96.18])
	by mail.kernel.org (Postfix) with ESMTP id 16AF723447
	for <patchwork-kvm@patchwork.kernel.org>;
 Fri, 18 Sep 2020 12:05:14 +0000 (UTC)
Authentication-Results: mail.kernel.org;
	dkim=pass (2048-bit key) header.d=gmail.com header.i=@gmail.com
 header.b="T9w+PwfL"
Received: (majordomo@vger.kernel.org) by vger.kernel.org via listexpand
        id S1726445AbgIRMFM (ORCPT
        <rfc822;patchwork-kvm@patchwork.kernel.org>);
        Fri, 18 Sep 2020 08:05:12 -0400
Received: from lindbergh.monkeyblade.net ([23.128.96.19]:33530 "EHLO
        lindbergh.monkeyblade.net" rhost-flags-OK-OK-OK-OK) by vger.kernel.org
        with ESMTP id S1726064AbgIRMFM (ORCPT <rfc822;kvm@vger.kernel.org>);
        Fri, 18 Sep 2020 08:05:12 -0400
Received: from mail-pg1-x544.google.com (mail-pg1-x544.google.com
 [IPv6:2607:f8b0:4864:20::544])
        by lindbergh.monkeyblade.net (Postfix) with ESMTPS id 26B39C06174A;
        Fri, 18 Sep 2020 05:05:12 -0700 (PDT)
Received: by mail-pg1-x544.google.com with SMTP id k14so3342998pgi.9;
        Fri, 18 Sep 2020 05:05:12 -0700 (PDT)
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=gmail.com; s=20161025;
        h=from:to:cc:subject:date:message-id:mime-version
         :content-transfer-encoding;
        bh=DF0IbIRrmgctFIOMM+pP2QIHakaraVxMAyB3OzkI7bk=;
        b=T9w+PwfLtu9hcKasjN4CLWsdksRHFlp5hTqZ5sYrKXoOOpQdPw3JhaqugmL257radv
         mkDK4j2QK/OtWBx8TibXQkNhdiTkZNZnoozHhcC5r4XBWWBZcNyqZkwf0xqpnOz+T14o
         Ytepacm13eQLSq9pl24RjwKsGdni50n4SCSbPG0E0WvvOmrjn3Md0rZrFcpC4dvKv78F
         8srlpkxOfHY41UHp6GVygbLRCxmJf38s/a5v+F7z6Ge9onhgpqtZsC3wJfV3a0g+rsi8
         +hTosIqDyDIfZAp0/Xm6FvQYzrST3759GRZ63BHMVEms5dKKyHWLAG9SZh6yBhZqmdCs
         atHA==
X-Google-DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=1e100.net; s=20161025;
        h=x-gm-message-state:from:to:cc:subject:date:message-id:mime-version
         :content-transfer-encoding;
        bh=DF0IbIRrmgctFIOMM+pP2QIHakaraVxMAyB3OzkI7bk=;
        b=cs8VSxSsQrnPkglNyCq/wo89ebqw8znDK7ePtChc6ULmIApNINPky26zpdlAh+0ZYh
         M3+xkBfR+G4vezmmKt6uEmYN72Pycq7pEQBMdz2vpv8q4vSYGZlCS8Doc0EKVkRClFsv
         QkvpzJqYKoyfJBFhFNuZCK0OkZOx1UkT+c9+KjObT9edB6uveD0BbFi4+o5HDGHRMK4r
         3k9Rd1+2bEQG2SRTlQqSf79eVccFzloR4Vz8+VxiJFXMuCKgSKq9Ufja6nHnJ63ODtP+
         LI5YQmFMtZPjTw8FE5s5N9shEJ4Td+TMBLUa02FIikfKXqYM2q+T0TeZAibGIIPI35VX
         V3cg==
X-Gm-Message-State: AOAM532Rb1vFtTVhdM7pOElUhy1mUehxWpZnOO5JZWY0fmHU2Pc4quwS
        1rclPfYeqIpKNtQ6mAF9nGg=
X-Google-Smtp-Source: 
 ABdhPJw3o9wEjT4p/RTkWgC91z3MI+qjjXL2Pvo+vpg8VXH//Dt0R2oGB033vcTS7fn9h0S4/V/fKg==
X-Received: by 2002:a63:4a19:: with SMTP id x25mr25782744pga.56.1600430711617;
        Fri, 18 Sep 2020 05:05:11 -0700 (PDT)
Received: from localhost.localdomain (104.36.148.139.aurocloud.com.
 [104.36.148.139])
        by smtp.gmail.com with ESMTPSA id
 n7sm2966274pfq.114.2020.09.18.05.05.07
        (version=TLS1_3 cipher=TLS_AES_256_GCM_SHA384 bits=256/256);
        Fri, 18 Sep 2020 05:05:11 -0700 (PDT)
From: Rustam Kovhaev <rkovhaev@gmail.com>
To: pbonzini@redhat.com, vkuznets@redhat.com, gustavoars@kernel.org,
        kvm@vger.kernel.org
Cc: linux-kernel@vger.kernel.org, gregkh@linuxfoundation.org,
        stable@vger.kernel.org, Rustam Kovhaev <rkovhaev@gmail.com>
Subject: [PATCH] KVM: use struct_size() and flex_array_size() helpers in
 kvm_io_bus_unregister_dev()
Date: Fri, 18 Sep 2020 05:05:00 -0700
Message-Id: <20200918120500.954436-1-rkovhaev@gmail.com>
X-Mailer: git-send-email 2.28.0
MIME-Version: 1.0
Precedence: bulk
List-ID: <kvm.vger.kernel.org>
X-Mailing-List: kvm@vger.kernel.org

Make use of the struct_size() helper to avoid any potential type
mistakes and protect against potential integer overflows
Make use of the flex_array_size() helper to calculate the size of a
flexible array member within an enclosing structure

Cc: stable@vger.kernel.org
Suggested-by: Gustavo A. R. Silva <gustavoars@kernel.org>
Signed-off-by: Rustam Kovhaev <rkovhaev@gmail.com>
---
 virt/kvm/kvm_main.c | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)

```
