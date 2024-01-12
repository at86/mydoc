# lambda

```c++
int main(){
  int a=2,b=2;
  int d=2;
	auto cb = [a,b,&d](){
    	int c=2;
      c+=a;
      c+=b;
      c+=d;
      return c;
    };
  cb();
}
```

https://cppinsights.io/lnk?code=aW50IG1haW4oKXsKICBpbnQgYT0yLGI9MjsKICBpbnQgZD0yOwoJYXV0byBjYiA9IFthLGIsJmRdKCl7CiAgICAJaW50IGM9MjsKICAgICAgYys9YTsKICAgICAgYys9YjsKICAgICAgYys9ZDsKICAgICAgcmV0dXJuIGM7CiAgICB9OwogIGNiKCk7Cn0=&insightsOptions=cpp23&std=cpp23&rev=1.0

# 直接初始化
```c++
int main(){
  struct A{
    int a;
  };
  A a(2);
}
```
