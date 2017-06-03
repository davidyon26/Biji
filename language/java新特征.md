1. static import
  * 静态引用（static import）允许在使用某个类的静态成员不需要每次都通过该类进行限定，而直接使用该静态成员，就好像这些静态成员是在使用它们的类中定义一样。在Effective Java中，建议使用继承达到该效果。
  * 用法：import **static** *class_full_name.static_memeber* 或者 *class_full_name.\**
  ```
  import static java.lang.Math.PI;
  或者
  import static java.lang.Math.*;
  这样如果想使用Math中的静态成员，就不需要通过Math进行限定，而直接使用其静态成员，如
  double r = cos(PI * theta);
  ```
  * 引入规范版本：Java 5
