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

------



#### 复杂匹配规则

**正则表达式：**

- **^(开头)和$(结尾)**匹配： `"^A\d{3}$"`  ->  "A001", "A748"
- **[...]** 可以匹配范围内的字符
  - `"[abc]1"`  ->  "a1", "b1", "c1"
  - `"[a-f]{3}"`  ->  "abc",  "fda"
  - `"[a-z0-9]{6}"`  ->  "_fffff",  "hd_83z"
- **[^...]** 可以匹配**非范围内的字符**： `"[^0-9]{3}"`  ->  "a@d", "jj-"
- **AB|CD** 可以**匹配AB或CD**： `"java|python"`  ->  "java", "python"
- **(AB|CD)**可以**匹配AB或CD**：  `"learn\s(java|python)"`  ->  "learn java", "learn python"



##### Code Demo

```java
package regular._basic;

public class QQ {
	
	/**
	 * 必须以非零数字开头的5至11位数的QQ号
	 */
	public static boolean isQQN(String qq) {
		return qq.matches("[1-9]\\d{4,10}");
	}
}
```

------



#### 分组匹配



**以Date为例**

`([0-2][0-9]{3})\\-(0[1-9]|1[0-2])\\-([1-9]|1[0-9]|2[0-9]|3[0-1])`

 

**正则表达式分组可以通过Matcher对象快速提取子串：**

- 以`()`来分组，按从左到右的顺序来提取子串

- `group(0)`表示匹配的整个字符串
- `group(1)`表示第一个子串
- `group(2)`表示第二个子串
- ....



##### Code Demo

```java
 package regular._basic;

import java.util.Objects;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class GroupRe {

    /**
     * start by 0, follow two to three numbers, -- first group
     * start by [1-9] and follow five to seven numbers
     */
	public static GroupRe parse(String s) {
		Pattern p = Pattern.compile("^(0\\d{2,3})\\-([1-9]\\d{5,7})$");
		Matcher m = p.matcher(s);
		if(m.matches()) {
			String s1 = m.group(1);
			String s2 = m.group(2);
			return new GroupRe(s1, s2);
		}
		return null;
	}
	
	private final String areaCode;
	private final String phone;
	

	public GroupRe(String areaCode, String phone) {
		this.areaCode = areaCode;
		this.phone = phone;
	}
	
	public String getAreaCode() {
		return areaCode;
	}

	public String getPhone() {
		return phone;
	}

	@Override
	public boolean equals(Object o) {
		if(o == this) {
			return true;
		}
		if(o instanceof GroupRe) {
			GroupRe g = (GroupRe) o;
			return Objects.equals(g.areaCode, this.areaCode) && Objects.equals(g.phone, this.phone);
		}
		return false;
	}
	
	@Override
	public int hashCode() {
		return Objects.hash(this.areaCode, this.phone);
	}
	
	@Override
	public String toString() {
		return this.areaCode + " - " + this.phone;
	}
}
```

------



#### 非贪婪匹配

 

例如：`^(\d+)(0*)$`  匹配 1230000，`(\d+)`即可匹配完整个数值，`(0*)`并未起到任何作用，即默认贪婪匹配

**正则表达式默认使用贪婪匹配**

- 使用`?`实现非贪婪匹配

####  

##### Code Demo

```java
package regular._basic;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class greedy {

	public static int zeros(String s) {
	    /**
	     * ^\\d+(0*)$ 不加?
	     * 贪婪匹配导致(0*)无匹配项
	     * group(1)内无返回值
	     */
		Pattern p = Pattern.compile("^\\d+?(0*)$");
		Matcher m = p.matcher(s);

		if (m.matches()) {
			String zeroStr = m.group(1);
			return zeroStr.length();
		}

		throw new IllegalArgumentException("Not a number");
	}
}
```

------



#### Search & Replace



**使用正则表达式分割字符串**

- `String[] String.split(String regex)`

**使用正则表达式搜索字符串**

- `Matcher.find()`

**使用正则表达式替换字符串**

- `String.replaceAll()`



##### Code Demo

Split

```java
package regular._basic;

import java.util.Arrays;

public class Split {

	public static void main(String[] args) {

		String tags = "java ,    ;  c++    python";
		/**
		 * 分割条件为：不等于数字和字母以及加号的一个或多个字符
		 */
		String[] arr = tags.split("[^0-9a-zA-Z\\+]+");
		System.out.println(Arrays.toString(arr)); // [java, c++, python]
		System.out.println(arr.length); // 3
	}
}
```

Search

```java
package regular._basic;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class Search {

	public static void main(String[] args) {
		String str = "Cookies allow us to distinguish you from other users of our website, personalise content and ads, provide social media features and analyse your use of this website. We may also share information about your usage with our social media, advertising and analytics partners. You may delete and block cookies from this website, but deleting some cookies may mean some parts of the website may not work";
        /**
         * 搜索条件为：包含c或C的单词
         */
		Pattern p = Pattern.compile("\\w*(c|C)\\w*");
		Matcher m = p.matcher(str);
	
        /**
         * print out start position and end position
         */
		while (m.find()) {
			String sub = str.substring(m.start(), m.end());
			System.out.println(sub + " : start=" + m.start() + ", end=" + m.end());
		}
	}
}

/**
 * result:
 * Cookies : start=0, end=7
 * content : start=81, end=88
 * social : start=106, end=112
 * social : start=222, end=228
 * analytics : start=252, end=261
 * block : start=291, end=296
 * cookies : start=297, end=304
 * cookies : start=342, end=349
 */
```

Replace

```java
package regular._basic;

public class Replace {

	public static void main(String[] args) {

		String s = "Cookies  allow  us  to  distinguish  you  from  other  users  of  our  website";
        /**
         * 用指定字符串替换包含t的字符串
         */
		String r = s.replaceAll("\\w*t\\w*", "T-word");
		System.out.println(r); // Cookies  allow  us  T-word  T-word  you  from  T-word  users  of  our  T-word
        
        /**
         * 用group的概念替换匹配项
         */
		String r2 = s.replaceAll("(\\w+)", "_$1_");
		System.out.println(r2); // _Cookies_  _allow_  _us_  _to_  _distinguish_  _you_  _from_  _other_  _users_  _of_  _our_  _website_
	}
}
```

