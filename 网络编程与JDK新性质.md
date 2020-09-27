## lambda表达式



```java
public class Test {
    public static void main(String[] args) {
        ArrayList<Student> list = new ArrayList<>();
        list.add(new Student("zhangsan",14,67));
        list.add(new Student("lisi",13,89));
        list.add(new Student("wangwu",15,97));
        list.add(new Student("maliu",12,63));
        list.add(new Student("zhaoqi",17,75));
        
        //需要新建一个AgeFilter方法
        getByFilter(list,new AgeFilter());
		
        //直接新建一个内部类filter
        getByFilter(list, new StudentFilter() {
            @Override
            public boolean compare(Student student) {
                return student.getScore()>80;
            }
        });
		//使用lambda表达式
        getByFilter(list,(a)->a.getName().length()>4);
        
    }
    public static void getByFilter(ArrayList<Student> list,StudentFilter filter){
        ArrayList<Student> list1 = new ArrayList<>();
        for(Student sti:list){
            if(filter.compare(sti)){
                list1.add(sti);
            }
        }
        printStudent(list1);
        System.out.println("---------------");
    }
    public static void printStudent(ArrayList<Student> students){
        for(Student student:students){
            System.out.println(student);
        }
    }
}
```

### 常用接口

![函数式接口](E:\deng\image\函数式接口.png)

