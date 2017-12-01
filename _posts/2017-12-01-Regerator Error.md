### 바벨의 regenerator는 무엇일까 ?

vuex에 async await를 적용하여 코드를 작성하니 아래와 같은 에러를 보게 되었다.

`Uncaught ReferenceError: regeneratorRuntime is not defined`

regenerator는 제너레이터 문법과 관련된 것 같은데 async, await는 내부적으로 제너레이터를 쓰니깐 에러는 맞는 것 같은데.. 

왜 에러가 나지? 난 이미 es6와 es7을 지원하는 babel-preset-env에 stage3까지 적용했는데 async await를 지원하지 않는단 말이야? 

라는 생각에 그래서 조금 더 파보게 되었다.





regenerator 모듈은 async와 제너레이터를 컴파일 시켜주는 역할을 하는 놈이다.

This plugin uses the regenerator module to transform async and generator functions. regeneratorRuntime is not included.

Runtime required
You need to use either the Babel polyfill or the regenerator runtime so that regeneratorRuntime will be defined.

Async functions
These are only usable if you enable their syntax plugin. See syntax-async-functions for information.

![Inline-image-2017-11-30 14.20.47.651.png](/files/2094763329077918158)


![Inline-image-2017-11-30 14.24.32.529.png](/files/2094765212703021132)

https://babeljs.io/docs/plugins/transform-runtime/

The runtime transformer plugin does three things:

Automatically requires babel-runtime/regenerator when you use generators/async functions.
Automatically requires babel-runtime/core-js and maps ES6 static methods and built-ins.
Removes the inline Babel helpers and uses the module babel-runtime/helpers instead.