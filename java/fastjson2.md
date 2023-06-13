```java
public class TestModel {
    private int age;
    private String name;
}
```

```java
String text = "{\"age\":18,\"name\":\"lunar\"}";

// 将JSON解析为JSONObject
JSONObject data = JSON.parseObject(text);

// 获取JSONObject属性
int age = data.getInteger("age");
String name = data.getString("name");
System.out.println(age + "," + name);

// JSONObject转化为JavaBean
TestModel testModel = data.getObject("key", TestModel.class);
System.out.println(testModel);
```

```java
String text = "[{\"age\":18,\"name\":\"lunar\"}, {\"age\":20,\"name\":\"james\"}]";

// 将JSON解析为JSONArray
JSONArray arr = JSON.parseArray(text);

// 获取JSONArray属性
System.out.println(arr.get(0));
for (int i = 0; i < arr.size(); i++) {
  JSONObject object = arr.getJSONObject(i);
  System.out.println(object.getInteger("age") + "," + object.getString("name"));
}

// JSONArray转JavaBean
List<TestModel> list = arr.toJavaList(TestModel.class);
System.out.println(list);
```

```java
// 将JSON解析为Java对象
String text = "{\"age\":18,\"name\":\"lunar\"}";
TestModel model = JSON.parseObject(text, TestModel.class);
System.out.println(model);

// 将JavaBean对象序列化为JSON
String str = JSON.toJSONString(model);
System.out.println(str);
```

