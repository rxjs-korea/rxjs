# 설치 지침

RxJS를 설치할 수 있는 다양한 방법:

## npm을 통한 ES2015

```shell
npm install rxjs
```

기본적으로, RxJS 7.x는 사용자에 따라 다양한 코드를 제공합니다:
* Node.js에서 RxJS 7.x를 사용하는 경우 `require` 또는 `import` 여부에 관계없이, ES5에서 실행할 수 있는 CommonJS 코드를 제공합니다.
* 브라우저(또는 기타 비 Node.js 플랫폼)를 대상으로 하는 번들러를 통해 RxJS 7.4+를 사용하는 경우 ES5에서 실행할 수 있는 ES 모듈 코드(ES2015 옵션 포함)를 기본적으로 제공됩니다.
7.4.0 이전의 7.x 버전은 ES5 코드만 제공합니다.

프로젝트의 목표 브라우저가 ES2015+를 지원하거나 번들 프로세스가 ES5의 하향 조정을 지원하는 경우 번들러를 선택적으로 구성하여 ES2015 RxJS 코드를 사용할 수 있습니다.
모듈 해석 중에 `es2015` 사용자 지정 내보내기를 사용하도록 번들러를 구성하여 ES2015 RxJS 코드 사용에 대한 지원을 활성화할 수 있습니다.
`es2015` 사용자 지정 내보내기를 사용하도록 번들러를 구성하는 것은 각 번들러에 따라 다릅니다.
이 옵션을 사용하는 데 관심이 있는 경우, 번들러 설명서에서 추가 정보를 참조하십시오.
하지만, 다음에서 몇 가지 일반적인 정보를 찾을 수 있습니다:

- https://webpack.js.org/guides/package-exports/#conditions-custom
- https://github.com/rollup/plugins/blob/node-resolve-v11.0.0/packages/node-resolve/README.md#exportconditions

필요한 것을 가져오고 싶다면, {@link guide/importing#es6-via-npm 해당} 가이드를 확인하세요.

## npm을 통한 CommonJS

에러 TS2304와 같은 오류가 발생하는 경우: 이름 'Promise'을 찾을 수 없거나 에러 TS2304인 경우: 
RxJS를 사용할 때 'Iterable'을 찾을 수 없습니다. 추가 typings 세트를 설치해야 할 수도 있습니다.

1.  typings 사용자인 경우:

```shell
typings install es6-shim --ambient
```

2.  typings을 사용하지 않는 경우에 /es6-shim/es6-shim.d.ts에서 인터페이스를 복사할 수 있습니다.

3.  tsconfig.json 또는 CLI 전달인자에 포함된 타입 정의 파일을 추가하세요.

## npm을 통한 모든 모듈 유형(CJS/ES6/AMD/TypeScript)

npm 버전 3을 통해 이 라이브러리를 설치하려면, 다음 명령을 사용하십시오:

```shell
npm install @reactivex/rxjs
```

npm 버전 2를 사용하는 경우, 라이브러리 버전을 명시적으로 지정해야 합니다:

```shell
npm install @reactivex/rxjs@7.3.0
```

## CDN

CDN의 경우, [unpkg](https://unpkg.com/)를 사용할 수 있습니다:

[https://unpkg.com/rxjs@^7/dist/bundles/rxjs.umd.min.js](https://unpkg.com/rxjs@%5E7/dist/bundles/rxjs.umd.min.js)

필요한 것을 가져오고 싶다면, {@link guide/importing#cdn 해당} 가이드를 확인하세요.
