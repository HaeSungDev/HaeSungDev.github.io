---
title:  "Spring Web MVC의 요청 처리 분석(DispatcherServlet 분석)"
search: true
categories: 
  - Spring
last_modified_at: 2021-02-07T10:40:00
---

웹 애플리케이션을 개발할 때 인증, 로깅, 뷰 템플릿 처리 등 많은 기능이 필요합니다. 이런 기능들을 직접 다 개발하면 생산성이 낮아지고 비즈니스 요구사항 반영에 집중하지 못하게 됩니다. Spring Web MVC는 이런 문제를 해결하기 위해 프레임워크 형태로 웹 개발에 필요한 다양한 기능들을 제공해줘서 개발자의 생산성을 크게 높여줍니다.

Spring Web MVC는 Servlet 기술을 기반으로 만들어졌고 Servlet을 이용해서 웹 요청을 처리하고 응답합니다. Spring Web MVC에서는 웹 요청을 DispatcherServlet이 전담해서 처리합니다. 따라서 Spring Web MVC의 요청 처리를 분석하는 것은 결국 DispathcherServlet을 분석하는 것입니다.

DispathcherServlet이 필요한 이유를 알아보고 구조를 분석해보도록 하겠습니다.

## DispatcherServlet이 필요한 이유

DispathcerServlet이 필요한 이유를 이해하려면 Servlet과 Front Controller 패턴이 무엇인지 알고있어야 합니다. 두 개념에 대해 간단히 알아보도록 하겠습니다.

### Servlet이란?

Servlet은 http 요청을 동적으로 처리하기 위한 Java EE의 스펙입니다. 요청을 동적으로 처리하는 코드를 Servlet 클래스를 스펙에 맞게 구현하고 Servlet 컨테이너에 등록하면 요청이 들어올 때 요청 Path에 해당하는 Servlet 클래스를 실행시켜줍니다.

Servlet을 활용하면 클라이언트의 요청에 따라 다양한 동작을 수행하는 코드를 작성할 수 있습니다. 여러개의 Servlet을 만들어서 다양한 요청을 처리할 수 있습니다. 그런데 웹에서는 보통 보안, 로깅 등의 공통적인 동작을 하는 코드가 포함 될 때가 많은데 Servlet을 여러개 만들다보면 공통 코드가 중복되는 일이 생길 수 있습니다.

이런 중복을 줄이기 위해 Front Controller라는 패턴이 나왔고 DispathcerServlet은 Front Controller 패턴의 구현체입니다.

### Front Controller 패턴이란?

Front Controller 패턴은 Servlet에서 공통되는 기능들을 통합하기 위한 패턴입니다. 하나의 Servlet 구현체만 Servlet Container에 등록해서 공통되는 기능들을 한번만 작성하고 Servlet 내부에서 path, header 등으로 요청 처리가 분기 되도록 구현합니다.

간단한 Front Controller 패턴의 예시입니다.

```
@WebServlet("/report")
public class MoodServlet extends HttpServlet {
  Map<String, HttpServlet> handlers;

  public void doGet(HttpServletRequest request, HttpServletResponse response) {
    String path = request.getPathInfo();

    logger.info("path: " + path);

    HttpServlet handler = handlers.get(path);
    if (path == null) {
      // 404
    }

    handler.doGet(request, response);
  }
}
```

위 예제에서는 get 요청시 공통 로깅 처리 후 handler map에서 처리 handler를 찾아서 요청 처리를 분기하는 코드입니다. 상용 웹 애플리케이션에서는 훨씬 많은 공통 기능이 들어 갈 수 있습니다.

Spring Web MVC의 DispathcherServlet에서는 웹 개발시 많이 쓰이는 수많은 공통 기능들을 미리 구현해놓았습니다.

정리하면 DispathcerServlet은 웹 개발시 많이 필요한 공통 기능들을 통합하는 Front Controller의 구현체라고 할 수 있습니다. 웹 개발시 필요한 기능들을 미리 제공해줘서 생산성있게 웹 애플리케이션을 빠르게 개발할 수 있도록 도와줍니다.

이제 DispathcherServlet이 어떤 구조로 되어있는지 분석해보도록 하겠습니다.

## DispathcerServlet 요청 처리 분석


## 참고

* [Front Controller Pattern - Martin Fowler](https://martinfowler.com/eaaCatalog/frontController.html)

* [Oracle Servlet](https://google.com)

* [Spring Web MVC DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)