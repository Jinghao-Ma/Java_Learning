## JUnit

[TOC]



**单元测试**：

- 单元测试是针对最小的功能单元编写测试代码
- Java程序的最小功能单元是方法（**function**）
- 单元测试就是针对单个Java方法的测试

**单元测试的好处**：

- 确保单个方法运行正常
- 如果修改了方法代码，只需确保其对应的单元测试通过
- 测试代码本身就可以作为示例代码
- 可以自动化运行所有测试并获得报告

**JUnit的特点：**

- 使用断言（**Assertion**）测试期望结果
- 可以方便地组织和运行测试
- 可以方便的查看测试结果
- 常用的IDE（例如Eclipse）都集成了JUnit
- 可以方便的集成到Maven

**JUnit的设计：**

- `TestCase`：一个`TestCase`表示一个测试
- `TestSuite`：一个`TestSuite`包含一组`TestCase`，表示一组测试
- `TestFixture`：一个`TestFixture`表示一个测试环境
- `TestResult`：用于收集测试结果
- `TestRunner`：用于运行测试
- `TestListener`：用于监听测试过程，收集测试数据
- `Assert`：用于断言测试结果是否正确

**使用Assert断言：**

- 断言相等：`assertEquals(100, x)`
- 断言数组相等：`assertArrayEquals({1, 2, 3}, x)`
- 断言浮点数相等：`assertEquals(3.1416, x, 0.0001)`
- 断言为`null`：`assertNull(x)`
- 断言为`true/false`：`assertTrue(x>0)  assertFalse(x<0)`
- 其他：`assertNotEquals() / assertNotNull()`



##### Code Demo

Main

```java
import java.util.Arrays;

public class Calculator_ {
    /*
     * 调用Calculator_类中的一个Function
    */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Calculator_ c = new Calculator_();
		int r = c.calculate("1+2+3");
		System.out.println(r);
		

	}
	
	public int calculate(String expression) {
		String[] ss = expression.split("\\+");
		System.out.println(expression + " => " + Arrays.toString(ss));
		int sum = 0;
		for(String s : ss) {
			sum = sum + Integer.parseInt(s.trim());
		}
		return sum;
	}

}
```

Test

```java
import static org.junit.Assert.*;

import org.junit.Test;

/*
 * 测试Calculator_类中的calculate()函数
*/

public class Calculator_Test {

	@Test
	public void test() {
		assertEquals(3, new Calculator_().calculate("1+2"));
		assertEquals(77, new Calculator_().calculate("33+44"));
	}
	
	@Test
	public void test1() {
		assertEquals(4, new Calculator_().calculate(" 1 + 3 "));
	}
}
```



- 一个`TestCase`包含一组相关的测试方法
- 使用Assert断言测试结果（注意浮点数`assertEquals`要指定`delta`）
- 每个测试方法必须为完全独立
- 测试代码必须非常简单
- 不能为测试代码再编写测试
- 测试需要覆盖各种输入条件，特别是边界条件



------



### After & Before



**同一个单元测试内的多个测试方法**

- 测试前都需要初始化某些对象
- 测试后可能需要清理资源



**JUnit使用`@Before`和`@After`：**

- 在`@Before`方法中初始化测试资源

  - `setUp()`

- 在`@After`方法中释放测试资源

  - `tearDown()`

  

**JUnit对应每个`@Test`方法：**

1. 实例化测试类

2. 执行`@Before`

3. 执行`@Test`

4. 执行`@After`

   

**使用`@Before`和`@Afte`r可以保证：**

- 单个`@Test`方法执行前会创建新的Test实例，实例变量的状态不会传递给下一个`@Test`方法
- 单个`@Test`方法前后会执行`@Before`和`@After`方法



**`@Before`和`@After`方法：**

- `@Before`方法初始化的对象存放在实例字段中
- 实例字段的状态不会影响下一个`@Test`

 

**`@BeforeClass`和`@AfterClass`静态方法：**

1. 在执行所有`@Test`方法前执行`@BeforeClass`静态方法
2. 执行所有测试
3. 在执行所有`@Test`方法后执行`@AfterClass`静态方法

 

- `@BeforeClass`静态方法初始化的对象只能存放在静态字段中
- 静态字段的状态会影响到所有的`@Test`



**初始化测试资源称为Fixture**

- `@Before`：初始化测试对象，例如：`input = new FileInputStream()`
- `@After`：销毁@Before创建的测试对象，例如：`input.close()`
- `@BeforeClass`：初始化非常耗时的资源，例如：创建数据库
- `@AfterClass`：清理`@BeforeClass`创建的资源，例如：删除数据库

 

##### Code Demo

Main

```java
import java.util.Arrays;

public class Calculator {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Calculator c = new Calculator();
		int r = c.calculate("1+2+3");
		System.out.println(r);
	}
	
	public int calculate(String exp) {
		String[] ss = exp.split("\\+");
		System.out.println(exp + " => " + Arrays.toString(ss));
		int sum = 0;
		
		for(String s : ss) {
			sum = sum + Integer.parseInt(s.trim());
		}
		return sum;
	}
}
```

Test

```java
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

class CalculatorTest {
	
	Calculator calc;
    /*
     * 初始化测试对象及相关测试前的操作
    */
	@BeforeEach
	public void setUp() throws Exception {
	    // 初始化测试对象 calc
		calc = new Calculator();
		System.out.println("Test Start");
	}

    /*
     * 清理@Before所创建的测试对象
    */
	@AfterEach
	void tearDown() throws Exception {
		System.out.println("Test finished");
	}

    /**
     * Several test methods
     */
	@Test
	public void test2Plus() {
		int r = calc.calculate("3+4");
		assertEquals(7, r);
	}
	
	@Test
	public void test2PlusB() {
		int r = calc.calculate("3 + 4 ");
		assertEquals(7, r);
	}
	
	@Test
	public void test3Plus() {
		int r = calc.calculate("3+4+8");
		assertEquals(15, r);
	}
	
	@Test
	public void test3PlusB() {
		int r = calc.calculate("3+ 4 + 8   ");
		assertEquals(15, r);
	}
}
```

B&A Logical

```java
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

class LogicalTest {

    /*
     * 初始化测试资源
    */
	@BeforeAll
	static void setUpBeforeClass() throws Exception {
		System.out.println("setUpBeforeClass:   Power on");
	}
    
    /*
     * 清理@BeforeClass创建的测试资源
    */
	@AfterAll
	static void tearDownAfterClass() throws Exception {
		System.out.println("tearDownAfterClass:   Power off");
	}

    /*
     * 初始化测试对象
    */
	@BeforeEach
	void setUp() throws Exception {
		System.out.println("setUp:   login");
	}

    /*
     * 清理@Before创建的测试对象
    */
	@AfterEach
	void tearDown() throws Exception {
		System.out.println("tearDown:   logout");
	}

	@Test
	void test() {
		System.out.println("test:   Working");
	}
}
```

Output

```shell
setUpBeforeClass:   Power on
setUp:   login
test:   Working
tearDown:   logout
tearDownAfterClass:   Power off
```



### Summary

- 理解JUnit执行测试的生命周期
- `@Before`用于初始化测试对象，测试对象以实例变量存放
- `@After`用于清理`@Before`创建的测试对象
- `@BeforeClass`用于初始化耗时资源，以静态变量存放
- `@AfterClass`用于清理`@BeforeClass`创建的资源