### go环境安装
 https://golang.org/dl/

### liteide客户端安装
 https://sourceforge.net/projects/liteide/files/

### go网上教程
  20小时入门 https://sourceforge.net/projects/liteide/files/

### go变量

```
package main //必须有一个main包
import "fmt" //导入包，必须使用
func test() (a, b, c int) {
	return 1, 2, 3
}
func main() {
	fmt.Println("hello go language,world")

	//变量格式   var 变量名  类型 ，在同一个{}中，变量唯一，变量声明必须使用
	// 没有初始化变量，默认为0，运行期间可以修改变量
	var a int
	fmt.Println("a = ", a)
	a = 178
	fmt.Println("a = ", a)      //自动加换行
	var b int = 10              //声明初始化并赋值
	b = 12                      //赋值
	c := 30                     //自动推倒类型，必须初始化，通过初始化确认类型
	fmt.Printf("%T,%T\n", c, b) //格式化输出
	d, e := 1, 2                //多重自动推动初始化
	d, e = e, d                 //交换数值
	fmt.Printf("d = %d，e=%d\n", d, e)
	var tmp int
	tmp, _ = d, e // _匿名变量，丢弃数据不处理，_匿名变量，配置函数返回值使用才有优势
	fmt.Println("tmp = ", tmp)
	_, d, e = test() //调用函数test,返回 1，2，3
	fmt.Printf("d = %d  e = %d \n", d, e)

	// 常量，不可修改  const 变量名  类型
	const a1 int = 10
	const b1 = 20 //自动推倒赋值
	fmt.Println("a = ", a)
	var ( //多变量自动推倒类型
		c1 = 1
		d1 = 1.2
	)
	const ( //多常量自动推动类型
		f1 = 1
		g1 = 1.2
	)
	//iota （枚举）常量自动生成器，一行累加1，iota遇到const,重置为0，可以只写一个iota,下面默认，同一行，值相同
	const (
		h1, i1 = iota, iota
		j1     = iota
	)
	fmt.Println(a1, b1, c1, d1, f1, g1, h1, i1, j1)
	str1 := "felix" //自动推导类型字符串
	fmt.Printf("str1 类型 %T\n", str1)
	
	t1 := 3.3 + 4.4i  //复数推倒类型
	fmt.Printf("%T\n",t1)
	fmt.Println(real(t1),imag(t1))  //打印出实数和虚数
}

```
