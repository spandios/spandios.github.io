---
layout: post
title: nestjs swagger문서에서 jwt사용해 리퀘스트 날리기
date: 2024-10-11 08:00 +0900
img_path: /assets/images/
category: sql
tags: [ DB,SQL, MYSQL ]
---

## NestJS Swagger 문서에서 JWT 사용해 리퀘스트 날리기

Swagger 문서를 보면 리퀘스트를 날릴 수 있는데, 우리의 API는 보통 토큰을 필요로 한다. 

Swagger에서 토큰을 매번 넣어주는게 귀찮았기 때문에 보통 리퀘스트 테스트는 Postman을 사용하고 Swagger는 API 문서를 보는데 사용하곤 했다.

근데 다시 조사해보니 토큰을 계속 저장할 수 있는 옵션이 있었다. 설정하는 방법을 알아보자.

### Swagger에서 JWT 설정하기

우선 `main.ts`에 아래와 같이 설정을 추가한다.

```typescript

  const config = new DocumentBuilder()
    .setTitle('TITLE')
    .setDescription('DESCRIPTION')
    .addBearerAuth(
      {
        type: 'http',
        scheme: 'bearer',
        bearerFormat: 'JWT',
        name: 'Authorization',
        description: 'Enter JWT token',
        in: 'header',
      },
      'access-token', // name
    )
    .build();

```

나는 `access-token`이라는 이름으로 토큰을 설정했다. 이제 Swagger 문서에 들어가보면 우측 상단에 `Authorize` 버튼이 생긴다. 

클릭하면 `access-token`을 입력할 수 있는 창이 나온다. 여기에 토큰을 입력하면 이제 Swagger에서도 토큰을 넣어서 리퀘스트를 날릴 수 있다.

여기서 추가적으로 `persistAuthorization`을 `true`로 설정하면 브라우저를 끄더라도 전에 저장한 토큰을 그대로 사용할 수 있다.

```typescript

 const swaggerCustomOptions: SwaggerCustomOptions = {
    swaggerOptions: {
      persistAuthorization: true,
    },
  };

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('docs', app, document, swaggerCustomOptions);


```



마지막으로 엔드포인트 중 토큰이 필요한 부분은 `@ApiBearerAuth({token-name})` 데코레이터를 추가해주면 된다. 반드시 `{token-name}`은 위에서 설정한 이름과 동일해야 한다.

```typescript

  @ApiBearerAuth('access-token')
  @Get()
  async getHello(@Request() req) {
    return req.user;
  }

```


## 주의 

만약 token name을 변경하면 전에 저장한 토큰이 사라지기 때문에 다시 입력해 저장하도록 하자. 나는 401 에러가 계속 떠서 당황했다 ㅠㅠ
















