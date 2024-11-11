









```dtd
/include/ "system-conf.dtsi"
/ {
        amba_pl: amba_pl{
                #address-cells = <1>;
                #size-cells = <1>;
                compatible = "simple-bus";
                ranges;
                irq: irq@0{
                        compatible = "hello,irq";
                        interrupt-parent = <&intc>;
                        interrupts = <0 29 2>;
                };
        };
};

/
{
   reserved-memory {
      #address-cells = <1>;
      #size-cells = <1>;
      ranges;

      reserved: buffer@0x10000000 {
         no-map;
         reg = <0x10000000 0x01000000>;
      };
   };

   reserved-driver@0 {
      compatible = "xlnx,reserved-memory";
      memory-region = <&reserved>;
   };
};
```

# Linux Reserved Memory 预留内存

### 预留内存给设备驱动

为了从系统地址空间预留内存，设备树须配置预留内存的节点。每个节点定义一个特定的内存空间，并且可以根据内核文档中关于可用于预留内存节点的说明配置不同的参数。然后就可以通过memory-region参数将预留的内存空间分配给特定的设备驱动程序使用。

用于64位Cortex-A53 MPSoC的system-top.dts文件中的设备树节点：

```dtd
 reserved-memory {
      #address-cells = <2>;
      #size-cells = <2>;
      ranges;

      reserved: buffer@0 {
         no-map;
         reg = <0x0 0x70000000 0x0 0x10000000>;
      };

   };

   reserved-driver@0 {
      compatible = "xlnx,reserved-memory";
      memory-region = <&reserved>;
   };
```



或32位Cortex-A9 Zynq上，最新的基于 Yocto的Petalinux 的自定义的类似的设备树节点：

```dtd
/include/ "system-conf.dtsi"
/
{
   reserved-memory {
      #address-cells = <1>;
      #size-cells = <1>;
      ranges;

      reserved: buffer0@0x10000000 {
         no-map;
         reg = <0x10000000 0x01000000>;
      };
   };

   reserved-driver@0 {
      compatible = "xlnx,reserved-memory";
      memory-region = <&reserved>;
   };
};
```







设备树参考文章：https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841683/Linux+Reserved+Memory















