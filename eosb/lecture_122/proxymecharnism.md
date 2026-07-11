## how to handle proxy in programatically

how to create proxy:
```
class SimpleClass{

    public void fun()
    {
        System.out.println("let,s fun with proxy ..........");
    }
}


class Main{
    public static void main(String[] args) {
        SimpleClass obj = new SimpleClass();
      ProxyFactory factory = new ProxyFactory(obj);
      factory.addAdvice(new MethodInterceptor() {
          @Override
          public @Nullable Object invoke(MethodInvocation invocation) throws Throwable {
              System.out.println("before execution of method is : "+ invocation.getMethod().getName());
              return invocation.proceed();
          }
      });
     SimpleClass proxyObj = (SimpleClass) factory.getProxy();
     proxyObj.fun();
        System.out.println("proxyObj is : "+proxyObj);
        System.out.println("simple class obj is "+ obj);
    }
}
```
