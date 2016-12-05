# Golang: can't load package: import cycle not allowed

> Date:  2018-06-11
>
> Author: sfwn

循环依赖:
`
A包(init.go) -> B包(func init() {}) -> A包(interface define)
`

A包 是接口，B包 是实现，然后 B包 里有 init() 函数，A 包用 init.go 来加载 B包 的 init() 函数，此时就出现了循环依赖。

稍微分析下，把 A 包 init.go 和 A包 里的接口 放在两个包里，那依赖关系就变成了:
1. 移动 A包 接口 到 A/interface 包:
    `A -> B -> A/interface`
2. 移动 A包 init.go 到 A/init 包:
    `A/init -> B -> A`
    
OK.