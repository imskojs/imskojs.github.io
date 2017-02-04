---
layout: post
title: Webpack HMR 개념
published: true
---

이 글은 Hot Module Replacement가 개념적으로 무엇인지 본다.

Hot Module Replacement (HMR) 은 app이 실행되있는 상황에서 module을 page refresh 없이 더하고, 빼고, 바꾸어주는 일을 한다.

## HMR 구동 원리.

### App을 구동하는 code 입장에서 볼때

1. app code는 HMR runtime에 update된 module이 있는지 물어본다.
2. HMR runtime은 update된 module이 있으면 download 받고 app code에게 새로운 updata가 있다고 알려준다.
3. app code는 HMR runtime에게 update를 적용하라고 한다.
4. HMR runtime은 update를 적용한다.


### Webpack 입장에서 볼때
Webpack은 update된것을 emit해야 합니다. (종종 bundle한 file을 내보내는 행위를 emit이라고 함)

Webpack이 update된것을 emit 할때 update가 2개의 part로 나누어 질수 있다.

1. update manifest (update 해야 하는 목록) (JSON).
2. update된 chunk들 (JavaScript).

update chunk들은 update된 module들을 가지고 있다.

### Module 입장에서 볼때
Hot Module Replacement (HMR) 는 HMR을 다루도록 코딩 되어 있는 module들에게만 적용 됩니다. 즉 특정 module이 HMR을 쓰지 않도록 coding되어 있으면 HMR을 못쓰는 것이지요.

예를 들어 `style-loader`는 HMR interface를 사용 합니다. `style-loader`는 HMR로 부터 "update"를 받으면 기존의 style을 새로운 style로 바꿉니다.

위와 비슷하게 HMR interface를 module에서 사용하면 여러분은 module이 update된것을 HMR에게 받으면 어떤일이 일어날지 정해줄수 있습니다.

만약 module에 HMR handler가 없다면 그 update는 bubble up됩니다. 즉, HMR interface를 module의 최상위에서 사용하면 update가 bubble up 된것을 handling할수 있습니다.


### HMR runtime 입장에서 볼때

HMR runtime은 `check`와 `apply` method를 사용합니다.

`check`는 update manifest에 HTTP request를 보냅니다. 만약 이 request가 fail하면 update가 없는것이고 succeed를 하면 update된 chunk들은 현제 load된 chunk들의 list와 비교를 하게 됩니다. 이미 load되어 있는 chunk별로 update chunk가 download됩니다. 모든 update module은 runtime에 저장 됩니다. 모든 update chunk가 download되고 적용할 준비가 되면 그때 runtime은 `ready` state로 변합니다.

`apply` method는 update module들을 전부 invalid로 flag합니다.
invalid로 flag된 module들은 update handler가 있거나 그 parent module에 update handler가 있어야합니다. 만약 없다면 이 invalid module들은 버려지게 되고 unload됩니다. update handler가 있는 module은 update되고 후 "accept" handler를 부르게 됩니다.

이후 runtime은 idle state가 되고 HMR와 관련된 작업은 전부 끝납니다.

## HMR로 무엇을 할수 있을까요?
특정 loader는 이미 hot-update할수 있는 기능을 가지고 있습니다. 예를 들어 `style-loader`는 다로 구현할필요없이 기본적으로 hot loading이 되는것이지요.

기본적으로 hot loadable이 세팅 되어 있지않는 module또는 loader를 하용할때는 `webpack-dev-server`를 이용하여 LiveReload를 대처할수도 있습니다. `webpack-dev-server`는 page reload전에 update를 HMR로 하려고 시도 합니다.


