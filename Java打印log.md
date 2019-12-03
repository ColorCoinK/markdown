---
layout: blog
title: Java打印log
date: 2019-07-25 14:45:45
categories:	
	- Blog
	- Java
tags:
	- Java
	- print
---
初学Java时的Log
<!--more-->

```java
/**
 * @Title: multiplicationTable
 * @Description: 九九乘法表
 * @ReturnType: void
 */
private void multiplicationTable() {
	// 乘法表
	for (int i = 1; i < 10; i++) {
		for (int j = 1; j <= i; j++) {
			System.out.print(j + " * " + i + " = " + (i * j) + "\t");
		}
		System.out.println();
	}
}

/**
 * @Title: rightAngle
 * @Description: 直角三角形
 * @param side
 * @ReturnType: void
 */
private void rightTriangle(int side) {
	// 直角三角形10
	for (int i = 0; i <= side; i++) {
		for (int j = i; j > 0; j--) {
			System.out.print("*");
		}
		System.out.println();
	}
	
}

/**
 * @Title: hollowrightTriangle
 * @Description: 镂空直角三角形
 * @param side
 * @ReturnType: void
 */
private void hollowRightTriangle(int side) {
	for (int i = 0; i <= side; i++) {
		for (int j = i; j > 0; j--) {
			if (j == 1 || j == i && i != side) {
				System.out.print("*");
			}
			if (i == side) {
				System.out.print("*");
			} else {
				System.out.print(" ");
			}
		}
		System.out.println();
	}
}

/**
 * @Title: Parallelogram
 * @Description: 平行四边形
 * @param side
 * @ReturnType: void
 */
private void parallelogram(int side) {
	for (int i = 0; i < side; i++) {
		for (int j = i + 1; j < side; j++) {
			System.out.print(" ");
		}
		for (int j = side; j > 0; j--) {
			System.out.print("*");
		}
		System.out.println();
	}
}

@Test
public void math() {
	System.out.println("---------------------九九乘法表");
	this.multiplicationTable();
	
	System.out.println("---------------------直角三角形");
	this.rightTriangle(10);
	
	System.out.println("---------------------镂空直角三角形");
	this.hollowRightTriangle(10);
	
	System.out.println("---------------------平行四边形");
	this.parallelogram(5);

    // 等边三角形


}
```

> 输出结果

```console
---------------------九九乘法表
1 * 1 = 1	
1 * 2 = 2	2 * 2 = 4	
1 * 3 = 3	2 * 3 = 6	3 * 3 = 9	
1 * 4 = 4	2 * 4 = 8	3 * 4 = 12	4 * 4 = 16	
1 * 5 = 5	2 * 5 = 10	3 * 5 = 15	4 * 5 = 20	5 * 5 = 25	
1 * 6 = 6	2 * 6 = 12	3 * 6 = 18	4 * 6 = 24	5 * 6 = 30	6 * 6 = 36	
1 * 7 = 7	2 * 7 = 14	3 * 7 = 21	4 * 7 = 28	5 * 7 = 35	6 * 7 = 42	7 * 7 = 49	
1 * 8 = 8	2 * 8 = 16	3 * 8 = 24	4 * 8 = 32	5 * 8 = 40	6 * 8 = 48	7 * 8 = 56	8 * 8 = 64	
1 * 9 = 9	2 * 9 = 18	3 * 9 = 27	4 * 9 = 36	5 * 9 = 45	6 * 9 = 54	7 * 9 = 63	8 * 9 = 72	9 * 9 = 81	
---------------------直角三角形

*
**
***
****
*****
******
*******
********
*********
**********
---------------------镂空直角三角形

* 
* * 
*  * 
*   * 
*    * 
*     * 
*      * 
*       * 
*        * 
***********
---------------------平行四边形
         **********
        **********
       **********
      **********
     **********
    **********
   **********
  **********
 **********
**********
```
