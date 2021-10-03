---
title: "Модули ядра VMware и Secure Boot"
date: 2021-10-03T19:22:33+03:00
draft: false
tags: ["vmware", "secure-boot"]
---

## tl;dr

Если вы, как и я, не отключали на своём компьютере Secure Boot перед установкой убунты, и если вы, как и я, хотите при этом использовать [VMware Player](https://www.vmware.com/products/workstation-player.html)/Workstation (дисклеймер: Workstation я не пробовал, смотрите сами, что там будет), сделайте следующее:

1. запустите VMware Player и получите окошко "Before you can run VMware several modules must be compiled and loaded into the running kernel";
2. нажмите на кнопку "build", подождите, пока сборка не окончится провалом (в появившемся окошке будет путь к логу);
3. убедитесь, что в логе никаких ошибок сборки на самом деле нет (warning'и будут, но это норма);
4. попробуйте самостоятельно загрузить модули `vmmon` и `vmnet` (и полюбуйтесь на ошибку "Operation not permitted") - при этом в `dmesg` будут сообщения вроде `Lockdown: modprobe: unsigned module loading is restricted; see man kernel_lockdown.7`;
5. посмотрите содержимое директории `/var/lib/shim-signed/mok`, в ней должны быть файлы `MOK.der` и `MOK.priv` (так в убунтах по умолчанию хранится Machine Owner Key - сертификат+ключ для подписи пользовательских модулей ядра - на других системах эти файлы могут лежать где-то ещё);
6. подпишите собранные модули этим ключом:

    ```bash
    $ sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 /var/lib/shim-signed/mok/MOK.priv /var/lib/shim-signed/mok/MOK.der $(modinfo -n vmmon)
    $ sudo /usr/src/linux-headers-$(uname -r)/scripts/sign-file sha256 /var/lib/shim-signed/mok/MOK.priv /var/lib/shim-signed/mok/MOK.der $(modinfo -n vmnet)
    ```

    (эти команды ничего не выведут в случае успеха);

7. ещё раз запустите VMware Player, убедитесь, что он работает;
8. перезагрузитесь для того, чтобы скрипты VMware настроили свои виртуальные сетевые адаптеры.

При этом нужно, чтобы заголовочные файлы текущего ядра были установлены, но они в любом случае нужны для сборки модулей.
