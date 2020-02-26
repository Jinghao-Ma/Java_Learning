## Regular Expression

------

[TOC]



------

### Summary

- 正则表达式是一个字符串
- 正则表达式用字符串描述一个匹配规则
- 使用正则表达式可以快速判断给定的字符串是否符合匹配规则
- Java内建正则表达式引擎`java.util.regex`

##### Code Demo

Main

```java
package regular._basic;

public class is1900s {
	
	public static boolean is19xx(String year) {
		if(year == null)
			return false;
		return year.matches("19\\d\\d");
	}
}
```

Test

```java
package regular._basic.test;

import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import regular._basic.is1900s;

class is1900sTest {

	@BeforeAll
	static void setUpBeforeClass() throws Exception {
	}

	@AfterAll
	static void tearDownAfterClass() throws Exception {
	}

	@BeforeEach
	void setUp() throws Exception {
	}

	@AfterEach
	void tearDown() throws Exception {
	}

	@Test
	void test() {
		assertTrue(is1900s.is19xx("1999"));
		assertTrue(is1900s.is19xx("1901"));
		assertTrue(is1900s.is19xx("1933"));
		assertTrue(is1900s.is19xx("1976"));
		assertTrue(is1900s.is19xx("1919"));
		
		assertFalse(is1900s.is19xx(null));	
	}
}
```

------



### Match rule

#### 正则表达式的编写

- 精确匹配

  - `"abc"`   ->  `"abc"`
  - `"a\&c"`  ->  `"a&c"`, 特殊字符需要转义
  - `"a\u548cc"`  ->  `"a和c"`，非ASCII字符用`\u####`表示

- 特殊符号

  - ###### `.`   可以匹配==**一个任意字符**==：`"a.c"  ->  "abc" , "a&c"`

  - ###### `\d` 可以匹配==**一个数字**==： `"00\d"  ->  "007" or "008"`

  - ###### `\w` 可以匹配==**一个字母、数字或下划线**==：`"java\w"  ->  "javac", "java9", "java_"`

  - ###### `\s` 可以匹配==**一个空白字符**==：  `"A\sB"  ->  "A B", "A   B"` *(tab也算是空白字符)*

  - ###### `\D` 可以匹配==**一个非数字**==：  `"00\D"  ->  "00a", "00#"`

  - ###### `\W` 可以匹配==**一个非字母，非数字或非下划线**==：`"java\W"  ->  "java#", "java ", "java!"`

  - ###### `\S` 可以匹配==**一个非空白字符**==：`"A\SB"  ->  "A&B",  "ABB", "A4B"`

  - ###### `*` 可以匹配==**任意个字符(零个或多个)**==：`"A\d*"  ->  "A",  "A5333"`

  - ###### `+` 可以匹配==**至少一个字符**==： `"A\d+"  ->  "A1", "A3435453"` 

  - ###### `?` 可以匹配==**零个或一个字符**==：  `"A\d?"  ->  "A",  "A7"`

  - ###### `{n}` 可以匹配==**n个字符**==：  `"\d{4}"  ->  "3456", "8453"`

  - ###### `{n,m}` 可以匹配==**n~m个字符**==：  `"\d{3,5}"  ->  "345", "2109", "78900"`

  - ###### `{n,}` 可以匹配==**至少n个字符**==：  `"\d{2,}"  ->  "23",  "233333333"`

  - ###### `{,m}` 可以匹配==**最多m个字符**==：  `"\d{,3}"  ->  "1", "23", "345"`



##### Code Demo

Main

```java
package regular._basic;

public class isValidNumber {
	
	public static boolean numberTel(String num) {
		return num.matches("0\\d{2,3}\\-\\d{6,8}");
	}
}
```

Test

```java
package regular._basic;

import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

class isValidNumberTest {

	@BeforeEach
	void setUp() throws Exception {
	}

	@AfterEach
	void tearDown() throws Exception {
	}

	@Test
	void testTrue() {
		assertTrue(isValidNumber.numberTel("0755-28429620"));
		assertTrue(isValidNumber.numberTel("010-1234567"));
	}
	
	@Test
	void testFalse() {
		assertFalse(isValidNumber.numberTel("123-6789068"));
	}
}
```

