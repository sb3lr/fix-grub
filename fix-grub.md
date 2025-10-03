# دليل شامل لإصلاح محمل الإقلاع GRUB

## 🚀 نظرة عامة

يُعد محمل الإقلاع **GRUB** (Grand Unified Bootloader) المكون الحيوي الذي يسمح لنظام التشغيل Linux بالبدء. عند تلفه أو الكتابة فوقه، يصبح النظام غير قابل للإقلاع. يوضح هذا الدليل كيفية استخدام بيئة إنقاذ (Live USB/CD) لإصلاح GRUB عبر عملية "حبس" النظام (**chroot**).

---

## 🛠️ الخطوات التفصيلية للإصلاح

**المتطلبات:**
* جهاز إقلاع حي (Live USB/CD) لنظام Linux.
* معرفة بأوامر سطر الأوامر الأساسية.

### المرحلة الأولى: التحضير وتركيب الأقسام (Mounting)

1.  **تحديد الأقسام:**
    استخدم `lsblk` أو `fdisk -l` لتحديد قسم الجذر (`/`) وقسم الإقلاع (`/boot` أو `EFI`).

2.  **تركيب القسم الجذري (Root Partition):**
    ```bash
    # استبدل sdXY بالمعرف الصحيح لقسم الجذر
    sudo mount /dev/sdXY /mnt
    ```

3.  **تركيب قسم الإقلاع (Boot Partition):**
    * **لأنظمة GPT/UEFI:**
        ```bash
        # استبدل sdXZ بقسم EFI (عادةً FAT32)
        sudo mount /dev/sdXZ /mnt/boot/efi
        ```
    * **لأنظمة MBR/BIOS:**
        ```bash
        # استبدل sdXZ بقسم /boot
        sudo mount /dev/sdXZ /mnt/boot
        ```

4.  **تركيب الأنظمة الزائفة (Pseudo-Filesystems):**
    يجب تركيب هذه المجلدات **باستخدام خيار `--bind`** داخل بيئة chroot:
    ```bash
    sudo mount --bind /proc /mnt/proc
    sudo mount --bind /sys /mnt/sys
    sudo mount --bind /dev /mnt/dev
    sudo mount --bind /dev/pts /mnt/dev/pts
    sudo mount --bind /run /mnt/run
    ```

5.  **(اختياري) تمكين الاتصال بالإنترنت:**
    لتمكين تحديث الحزم داخل بيئة chroot:
    ```bash
    sudo cp /etc/resolv.conf /mnt/etc/
    ```

---

### المرحلة الثانية: الدخول إلى Chroot والإصلاح

6.  **الدخول إلى بيئة الحبس (`chroot`):**
    ```bash
    sudo chroot /mnt
    ```

7.  **التحديث والتأكد من الحزم:**
    تأكد من تحديث النظام وتثبيت حزم GRUB الصحيحة (مثل `grub-efi` أو `grub-pc`):
    ```bash
    apt update && apt upgrade -y # لمستخدمي دبيان/أوبونتو
    # أو
    dnf update # لمستخدمي فيدورا/سينت أو إس
    ```

8.  **تثبيت GRUB وإعادة إنشائه:**

    * **تثبيت GRUB:**
        * **لأنظمة UEFI:**
            ```bash
            grub-install --efi-directory=/boot/efi
            ```
        * **لأنظمة BIOS/MBR:**
            ```bash
            # استبدل sdX بالقرص الصلب بأكمله (مثال: /dev/sda)
            grub-install /dev/sdX
            ```

    * **إنشاء ملف إعدادات GRUB (مهم!):**
        ```bash
        grub-mkconfig -o /boot/grub/grub.cfg
        ```
        > **ملاحظة:** تأكد من أن المسار `grub.cfg` صحيح. بعض التوزيعات قد تستخدم مسارًا مختلفًا.

---

### المرحلة الثالثة: الإنهاء والاحتياطات

9.  **الخروج وإلغاء التركيب (Unmounting):**
    ```bash
    exit
    sudo umount -R /mnt
    ```
    الآن يمكنك إعادة تشغيل الجهاز.

10. **دعاء وتهيئة الإقلاع (Debugging):**
    * **8 في الأصل:** **ادعي ربك انه يشتغل!**
    * (if not. then the laptop is a POS (PIECE OF SHIT))
    * **في حال الفشل:** قد تحتاج إلى استخدام أدوات خارجية مثل **Hiren's BootCD** (باستخدام أدوات إدارة الإقلاع مثل EasyBCD أو ما يماثلها).
    * **إذا لم يظهر مدخل الإقلاع:** قد تحتاج إلى الدخول إلى إعدادات **BIOS/UEFI** وإضافة مدخل الإقلاع يدويًا، مشيرًا إلى ملف GRUB في قسم EFI. (هذه الميزة غير متوفرة في جميع الأجهزة).

> **إخلاء مسؤولية:** هذه العملية قد تعرض بياناتك للخطر إذا لم تُنفذ بدقة. يجب المضي قدمًا بحذر شديد.
