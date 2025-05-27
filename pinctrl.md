<!--
 * @Date: 2025-05-27
 * @LastEditors: Goko Son
 * @LastEditTime: 2025-05-27
 * @FilePath: /pinctrl/pinctrl.md
 * @Description: 
-->
1. 将pinctrl和clk都设置为M模式编译
2. 在initramfs的init中insmod
```
/ # cat init
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys

insmod /lib/modules/pinctrl-k1.ko
insmod /lib/modules/spacemit-ccu-k1.ko

exec /bin/sh </dev/console >/dev/console 2>&1
```
3. 启动之后挂载debugfs
```
mount -t debugfs debugfs /sys/kernel/debug
```
4. 运行insmod和rmmod命令
```
/ # mount -t debugfs debugfs /sys/kernel/debug
/ # ls sys/kernel/debug/
asoc                   extfrag                pm_genpd
bdi                    fault_around_bytes     regmap
block                  gpio                   regulator
btt                    hid                    sched
clear_warn_once        i2c                    slab
clk                    iio                    sleep_time
devfreq                memblock               stackdepot
device_component       mtd                    suspend_stats
devices_deferred       multigrain_timestamps  swiotlb
dma_buf                opp                    usb
dma_pools              phy                    virtio-ports
dmaengine              pinctrl                wakeup_sources
/ # lsmod
spacemit_ccu_k1 61440 0 - Live 0xffffffff0179d000
pinctrl_k1 20480 2 [permanent], Live 0xffffffff0178f000
/ # rmmod pinctrl_k1
rmmod: remove 'pinctrl_k1': Device or resource busy
/ # rmmod spacemit_ccu_k1
[   73.201767] clk_unregister: unregistering prepared clock: cpu_c1_ace_clk
[   73.206009] clk_unregister: unregistering prepared clock: cpu_c1_core_clk
[   73.212746] clk_unregister: unregistering prepared clock: cpu_c1_hi_clk
[   73.219392] clk_unregister: unregistering prepared clock: cpu_c0_tcm_clk
[   73.226019] clk_unregister: unregistering prepared clock: cpu_c0_ace_clk
[   73.232704] clk_unregister: unregistering prepared clock: cpu_c0_core_clk
[   73.239460] clk_unregister: unregistering prepared clock: cpu_c0_hi_clk
[   73.246075] clk_unregister: unregistering prepared clock: cci550_clk
[   73.253445] clk_unregister: unregistering prepared clock: pll3_d1
[   73.259817] ------------[ cut here ]------------
[   73.259828] WARNING: CPU: 3 PID: 111 at drivers/clk/clk.c:4497 clk_nodrv_disable_unprepare+0x8/0x12
[   73.259857] Modules linked in: spacemit_ccu_k1(-) pinctrl_k1
[   73.259875] CPU: 3 UID: 0 PID: 111 Comm: rmmod Not tainted 6.15.0-rc7-ga81e750e716d-dirty #3 NONE
[   73.259885] Hardware name: Banana Pi BPI-F3 (DT)
[   73.259889] epc : clk_nodrv_disable_unprepare+0x8/0x12
[   73.259897]  ra : clk_core_disable+0x42/0x80
[   73.259907] epc : ffffffff80589054 ra : ffffffff80589662 sp : ffffffc600503c00
[   73.259912]  gp : ffffffff81716fd0 tp : ffffffd7018b8000 t0 : ffffffc600503928
[   73.259917]  t1 : 00000000002a0018 t2 : 6765726e755f6b6c s0 : ffffffc600503c10
[   73.259922]  s1 : ffffffd70230a600 a0 : ffffffff017a6638 a1 : 0000000000000018
[   73.259927]  a2 : 0000000000000001 a3 : 0000000000ff00ff a4 : 0000000000000000
[   73.259931]  a5 : ffffffff8058904c a6 : ffffffd7000a4a80 a7 : 000000000000003f
[   73.259936]  s2 : ffffffd70230a600 s3 : 0000000000000000 s4 : 0000000200000022
[   73.259939]  s5 : ffffffd70230a600 s6 : ffffffd70087daf8 s7 : 00000000000101e0
[   73.259943]  s8 : 000000000000003a s9 : 00000000001cf2d0 s10: ffffffffffffffff
[   73.259947]  s11: 00000000001d1a18 t3 : ffffffff8172ba6f t4 : ffffffff8172ba6f
[   73.259952]  t5 : ffffffff8172ba70 t6 : ffffffc600503ad8
[   73.259955] status: 0000000200000100 badaddr: 0000000000000000 cause: 0000000000000003
[   73.259961] [<ffffffff80589054>] clk_nodrv_disable_unprepare+0x8/0x12
[   73.259970] [<ffffffff80589662>] clk_core_disable+0x42/0x80
[   73.259978] [<ffffffff8058b430>] __clk_set_parent_after+0x52/0xc0
[   73.259986] [<ffffffff8058c5aa>] clk_core_set_parent_nolock+0xae/0x186
[   73.259992] [<ffffffff8058c74a>] clk_unregister+0xb0/0x1e6
[   73.259999] [<ffffffff8058c8a6>] devm_clk_hw_unregister_cb+0x10/0x18
[   73.260005] [<ffffffff8061846a>] devres_release_all+0x80/0xd4
[   73.260015] [<ffffffff806129a4>] device_unbind_cleanup+0x10/0x4a
[   73.260024] [<ffffffff80613a72>] device_release_driver_internal+0x184/0x1b4
[   73.260032] [<ffffffff80613b0c>] driver_detach+0x3a/0x80
[   73.260038] [<ffffffff80612520>] bus_remove_driver+0x50/0x9e
[   73.260045] [<ffffffff8061418e>] driver_unregister+0x22/0x48
[   73.260054] [<ffffffff806152b0>] platform_driver_unregister+0x10/0x18
[   73.260064] [<ffffffff0179dcb8>] k1_ccu_driver_exit+0x18/0x360 [spacemit_ccu_k1]
[   73.260076] [<ffffffff800a04cc>] __riscv_sys_delete_module+0x154/0x1e0
[   73.260088] [<ffffffff80a068fe>] do_trap_ecall_u+0x186/0x206
[   73.260100] [<ffffffff80a110ea>] handle_exception+0x146/0x152
[   73.260109] ---[ end trace 0000000000000000 ]---
[   73.494620] clk_unregister: unregistering prepared clock: pll3
[   73.499545] clk_unregister: unregistering protected clock: pll3
[   73.507410] clk_unregister: unregistering prepared clock: pll1_d5_491p52
[   73.519243] clk_unregister: unregistering prepared clock: uart0_bus_clk
[   73.527740] clk_unregister: unregistering prepared clock: uart0_clk
/ # insmod /lib/modules/spacemit-ccu-k1.ko
/ # lsmod
spacemit_ccu_k1 61440 0 - Live 0xffffffff01797000
pinctrl_k1 20480 2 [permanent], Live 0xffffffff0178f000
/ # rmmod spacemit_ccu_k1
[  127.941173] clk_unregister: unregistering prepared clock: cpu_c1_ace_clk
[  127.945408] clk_unregister: unregistering prepared clock: cpu_c1_core_clk
[  127.952159] clk_unregister: unregistering prepared clock: cpu_c1_hi_clk
[  127.958858] clk_unregister: unregistering prepared clock: cpu_c0_tcm_clk
[  127.965437] clk_unregister: unregistering prepared clock: cpu_c0_ace_clk
[  127.972116] clk_unregister: unregistering prepared clock: cpu_c0_core_clk
[  127.979667] clk_unregister: unregistering prepared clock: cpu_c0_hi_clk
[  127.985522] clk_unregister: unregistering prepared clock: cci550_clk
[  127.992901] clk_unregister: unregistering prepared clock: pll3_d1
[  128.000088] clk_unregister: unregistering prepared clock: pll3
[  128.003590] clk_unregister: unregistering protected clock: pll3
[  128.011435] clk_unregister: unregistering prepared clock: pll1_d5_491p52
[  128.022017] clk_unregister: unregistering prepared clock: uart0_bus_clk
[  128.030552] clk_unregister: unregistering prepared clock: uart0_clk
/ #

```
- pinctrl不能rmmod,显示busy，是串口在使用or...
- clk启动之后的第一次rmmod有警告，但是第二次就没有了
