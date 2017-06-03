## static import
  * 引入规范版本：Java 5
  * 静态引用（static import）允许在使用某个类的静态成员不需要每次都通过该类进行限定，而直接使用该静态成员，就好像这些静态成员是在使用它们的类中定义一样。在Effective Java中，建议使用继承达到该效果。
  * 用法：import **static** *class_full_name.static_memeber* 或者 *class_full_name.\**
  ```
  import static java.lang.Math.PI;
  或者
  import static java.lang.Math.*;
  
  如果想使用Math中的静态成员，就不需要通过Math进行限定，而直接使用其静态成员，如
  double r = cos(PI * theta);
  ```
## interface default and static method
  * 引入规范版本：Jave 8
  * 默认方法（**default method**）允许一个接口增加抽象方法而保证向后兼容性。即一个接口增加新的提供默认实现（defualt）方法后，原来实现旧版本接口的实现类不需要重新改写且继承了新接口定义的默认实现
  * 语法：在方法前使用`default`修饰符
  * 用例：
  ```
  package defaultmethods;
 
  import java.time.*;

  public interface TimeClient {
    void setTime(int hour, int minute, int second);
    void setDate(int day, int month, int year);
    void setDateAndTime(int day, int month, int year,
                               int hour, int minute, int second);
    LocalDateTime getLocalDateTime();
    
    static ZoneId getZoneId (String zoneString) {
        try {
            return ZoneId.of(zoneString);
        } catch (DateTimeException e) {
            System.err.println("Invalid time zone: " + zoneString +
                "; using default time zone instead.");
            return ZoneId.systemDefault();
        }
    }
        
    default ZonedDateTime getZonedDateTime(String zoneString) {
        return ZonedDateTime.of(getLocalDateTime(), getZoneId(zoneString));
    }
  }
  ```
  
