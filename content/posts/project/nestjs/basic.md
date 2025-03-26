+++
title = 'Nestjs 핵심만'
date = 2024-10-08T00:20:22+09:00
draft = true
+++
## Nestjs 핵심만


🟡 Nestjs Validation Pipe
- main에서 whitelist: true,
- dto에서 객체로 오는 값의 경우 ValidateNested 꼭 붙이고
    @IsObject()
    @ValidateNested()
    @Type(() => Content)
- class Content 안에 @IsString(), @Optional()등 하나하나 검증해줘야함.

---

@Res를 쓴 순간 직접 res.send를 해야하는데
passthrough를 하면 setcookie 외에는 알아서 해줘라- 

근데 return을 해놓고 passthrough를 하면 
또 무언가를 프레임워크단에서 res send 시키려고 하니까 문제생김

---