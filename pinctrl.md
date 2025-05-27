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

或许就是pinctrl被d4017000.serial使用了：
```/ # cat sys/kernel/debug/pinctrl/d401e000.pinctrl/pinmux-pins
Pinmux settings per pin
Format: pin (name): mux_owner|gpio_owner (strict) hog?
pin 0 (GPIO_00): UNCLAIMED
pin 1 (GPIO_01): UNCLAIMED
pin 2 (GPIO_02): UNCLAIMED
pin 3 (GPIO_03): UNCLAIMED
pin 4 (GPIO_04): UNCLAIMED
pin 5 (GPIO_05): UNCLAIMED
pin 6 (GPIO_06): UNCLAIMED
pin 7 (GPIO_07): UNCLAIMED
pin 8 (GPIO_08): UNCLAIMED
pin 9 (GPIO_09): UNCLAIMED
pin 10 (GPIO_10): UNCLAIMED
pin 11 (GPIO_11): UNCLAIMED
pin 12 (GPIO_12): UNCLAIMED
pin 13 (GPIO_13): UNCLAIMED
pin 14 (GPIO_14): UNCLAIMED
pin 15 (GPIO_15): UNCLAIMED
pin 16 (GPIO_16): UNCLAIMED
pin 17 (GPIO_17): UNCLAIMED
pin 18 (GPIO_18): UNCLAIMED
pin 19 (GPIO_19): UNCLAIMED
pin 20 (GPIO_20): UNCLAIMED
pin 21 (GPIO_21): UNCLAIMED
pin 22 (GPIO_22): UNCLAIMED
pin 23 (GPIO_23): UNCLAIMED
pin 24 (GPIO_24): UNCLAIMED
pin 25 (GPIO_25): UNCLAIMED
pin 26 (GPIO_26): UNCLAIMED
pin 27 (GPIO_27): UNCLAIMED
pin 28 (GPIO_28): UNCLAIMED
pin 29 (GPIO_29): UNCLAIMED
pin 30 (GPIO_30): UNCLAIMED
pin 31 (GPIO_31): UNCLAIMED
pin 32 (GPIO_32): UNCLAIMED
pin 33 (GPIO_33): UNCLAIMED
pin 34 (GPIO_34): UNCLAIMED
pin 35 (GPIO_35): UNCLAIMED
pin 36 (GPIO_36): UNCLAIMED
pin 37 (GPIO_37): UNCLAIMED
pin 38 (GPIO_38): UNCLAIMED
pin 39 (GPIO_39): UNCLAIMED
pin 40 (GPIO_40): UNCLAIMED
pin 41 (GPIO_41): UNCLAIMED
pin 42 (GPIO_42): UNCLAIMED
pin 43 (GPIO_43): UNCLAIMED
pin 44 (GPIO_44): UNCLAIMED
pin 45 (GPIO_45): UNCLAIMED
pin 46 (GPIO_46): UNCLAIMED
pin 47 (GPIO_47): UNCLAIMED
pin 48 (GPIO_48): UNCLAIMED
pin 49 (GPIO_49): UNCLAIMED
pin 50 (GPIO_50): UNCLAIMED
pin 51 (GPIO_51): UNCLAIMED
pin 52 (GPIO_52): UNCLAIMED
pin 53 (GPIO_53): UNCLAIMED
pin 54 (GPIO_54): UNCLAIMED
pin 55 (GPIO_55): UNCLAIMED
pin 56 (GPIO_56): UNCLAIMED
pin 57 (GPIO_57): UNCLAIMED
pin 58 (GPIO_58): UNCLAIMED
pin 59 (GPIO_59): UNCLAIMED
pin 60 (GPIO_60): UNCLAIMED
pin 61 (GPIO_61): UNCLAIMED
pin 62 (GPIO_62): UNCLAIMED
pin 63 (GPIO_63): UNCLAIMED
pin 64 (GPIO_64): UNCLAIMED
pin 65 (GPIO_65): UNCLAIMED
pin 66 (GPIO_66): UNCLAIMED
pin 67 (GPIO_67): UNCLAIMED
pin 68 (GPIO_68): device d4017000.serial function uart0-2-cfg group uart0-2-cfg.uart0-2-pins
pin 69 (GPIO_69): device d4017000.serial function uart0-2-cfg group uart0-2-cfg.uart0-2-pins
pin 70 (GPIO_70/PRI_DTI): UNCLAIMED
pin 71 (GPIO_71/PRI_TMS): UNCLAIMED
pin 72 (GPIO_72/PRI_TCK): UNCLAIMED
pin 73 (GPIO_73/PRI_TDO): UNCLAIMED
pin 74 (GPIO_74): UNCLAIMED
pin 75 (GPIO_75): UNCLAIMED
pin 76 (GPIO_76): UNCLAIMED
pin 77 (GPIO_77): UNCLAIMED
pin 78 (GPIO_78): UNCLAIMED
pin 79 (GPIO_79): UNCLAIMED
pin 80 (GPIO_80): UNCLAIMED
pin 81 (GPIO_81): UNCLAIMED
pin 82 (GPIO_82): UNCLAIMED
pin 83 (GPIO_83): UNCLAIMED
pin 84 (GPIO_84): UNCLAIMED
pin 85 (GPIO_85): UNCLAIMED
pin 86 (GPIO_86): UNCLAIMED
pin 87 (GPIO_87): UNCLAIMED
pin 88 (GPIO_88): UNCLAIMED
pin 89 (GPIO_89): UNCLAIMED
pin 90 (GPIO_90): UNCLAIMED
pin 91 (GPIO_91): UNCLAIMED
pin 92 (GPIO_92): UNCLAIMED
pin 93 (GPIO_93/PWR_SCL): UNCLAIMED
pin 94 (GPIO_94/PWR_SDA): UNCLAIMED
pin 95 (GPIO_95/VCX0_EN): UNCLAIMED
pin 96 (GPIO_96/DVL0): UNCLAIMED
pin 97 (GPIO_97/DVL1): UNCLAIMED
pin 98 (GPIO_98/QSPI_DAT3): UNCLAIMED
pin 99 (GPIO_99/QSPI_DAT2): UNCLAIMED
pin 100 (GPIO_100/QSPI_DAT1): UNCLAIMED
pin 101 (GPIO_101/QSPI_DAT0): UNCLAIMED
pin 102 (GPIO_102/QSPI_CLK): UNCLAIMED
pin 103 (GPIO_103/QSPI_CS1): UNCLAIMED
pin 104 (GPIO_104/MMC1_DAT3): UNCLAIMED
pin 105 (GPIO_105/MMC1_DAT2): UNCLAIMED
pin 106 (GPIO_106/MMC1_DAT1): UNCLAIMED
pin 107 (GPIO_107/MMC1_DAT0): UNCLAIMED
pin 108 (GPIO_108/MMC1_CMD): UNCLAIMED
pin 109 (GPIO_109/MMC1_CLK): UNCLAIMED
pin 110 (GPIO_110): UNCLAIMED
pin 111 (GPIO_111): UNCLAIMED
pin 112 (GPIO_112): UNCLAIMED
pin 113 (GPIO_113): UNCLAIMED
pin 114 (GPIO_114): UNCLAIMED
pin 115 (GPIO_115): UNCLAIMED
pin 116 (GPIO_116): UNCLAIMED
pin 117 (GPIO_117): UNCLAIMED
pin 118 (GPIO_118): UNCLAIMED
pin 119 (GPIO_119): UNCLAIMED
pin 120 (GPIO_120): UNCLAIMED
pin 121 (GPIO_121): UNCLAIMED
pin 122 (GPIO_122): UNCLAIMED
pin 123 (GPIO_123): UNCLAIMED
pin 124 (GPIO_124): UNCLAIMED
pin 125 (GPIO_125): UNCLAIMED
pin 126 (GPIO_126): UNCLAIMED
pin 127 (GPIO_127): UNCLAIMED
```