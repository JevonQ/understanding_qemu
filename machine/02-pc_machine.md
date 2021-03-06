相对MachineType， PCMachine就要复杂得多了。

还是按照之前的顺序，先来看继承关系。

# 继承关系

```
   TYPE_OBJECT
   +-------------------------------+
   |abstract                       | = true
   |class_init                     | = object_class_init
   |                               |
   |instance_size                  | = sizeof(Object)
   +-------------------------------+


   TYPE_MACHINE
   +-------------------------------+
   |abstract                       | = true
   |class_size                     | = sizeof(MachineClass)
   |class_init                     | = machine_class_init
   |class_base_init                | = machine_class_base_init
   |                               |
   |instance_size                  | = sizeof(MachineState)
   |instance_init                  | = machine_initfn
   |instance_finalize              | = machine_finalize
   +-------------------------------+


   TYPE_PC_MACHINE
   +-------------------------------+
   |abstract                       | = true
   |class_size                     | = sizeof(PCMachineClass)
   |class_init                     | = pc_machine_class_init
   |class_base_init                | = NULL
   |                               |
   |instance_size                  | = sizeof(PCMachineState)
   |instance_init                  | = pc_machine_initfn
   |instance_finalize              | = NULL
   +-------------------------------+


   pc_i440x_4.0-machine
   +-------------------------------+
   |abstract                       | = true
   |class_init                     | = pc_machine_v4_0_class_init
   |                               |       -> pc_i440fx_4_0_machine_options
   |                               |
   |instance_size                  | = sizeof(PCMachineState)
   |instance_init                  | = pc_machine_initfn
   |instance_finalize              | = NULL
   |                               |
   |init                           | = pc_init_v4_0
   |                               |       -> pc_init1
   +-------------------------------+      
```

所以我们看到PCMachine只是一个抽象父类，这正的虚拟机是它的子类。

而这些子类都由一个宏来定义

```
#define DEFINE_PC_MACHINE(suffix, namestr, initfn, optsfn) \
    static void pc_machine_##suffix##_class_init(ObjectClass *oc, void *data) \
    { \
        MachineClass *mc = MACHINE_CLASS(oc); \
        optsfn(mc); \
        mc->init = initfn; \
    } \
    static const TypeInfo pc_machine_type_##suffix = { \
        .name       = namestr TYPE_MACHINE_SUFFIX, \
        .parent     = TYPE_PC_MACHINE, \
        .class_init = pc_machine_##suffix##_class_init, \
    }; \
    static void pc_machine_init_##suffix(void) \
    { \
        type_register(&pc_machine_type_##suffix); \
    } \
    type_init(pc_machine_init_##suffix)
```

其中重要的就是将mc->init赋值。也就是在machine_run_board_init()函数中调用的部分。

# 初始化

初始化的工作很大一部分由mc->init函数完成，对于piix机器，这个工作就交给了pc_init1函数。

具体的我就不展开了，留到需要的时候再做讲解。
