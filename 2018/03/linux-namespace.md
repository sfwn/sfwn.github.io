# linux namespace

> Date: 2018-03-28
>
> Author: sfwn

当前Linux一共实现六种不同类型的namespace。


|Namespace类型|系统调用参数|内核版本|
|------------|:---------|------:|
|Mount namespaces | CLONE_NEWNS | 2.4.19 |
|UTS namespaces | CLONE_NEWUTS | 2.6.19 |
|IPC namespaces | CLONE_NEWIPC | 2.6.19 |
|PID namespaces | CLONE_NEWPID | 2.6.24 |
|Network namespaces | CLONE_NEWNET | 2.6.29 |
|User namespaces | CLONE_NEWUSER | 3.8 |