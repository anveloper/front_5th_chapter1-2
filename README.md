## 과제 체크포인트
### 배포 링크

- https://anveloper.github.io/front_5th_chapter1-2/profile
  - /login/index.html, /profile/index.html SPA용 SSG fallback 설정
  - createRouter.js > getPath 시 BASE_URL 생략하기
  - github pages 특성 상 /profile/index.html > /profile/ 로 trailing slash routing 현상 발생
    ```javascript
    const getPath = () => {
      let path = window.location.pathname;
      if (!!BASE_URL && path.startsWith(BASE_URL)) path = path.slice(BASE_URL.length - 1);
      if (path.length > 1 && path.endsWith("/")) path = path.slice(0, -1);
      return path;
    }; 
    ```

### 기본과제
<details><summary>1) 가상돔을 기반으로 렌더링하기</summary>

- [x] createVNode 함수를 이용하여 vNode를 만든다.
- [x] normalizeVNode 함수를 이용하여 vNode를 정규화한다.
- [x] createElement 함수를 이용하여 vNode를 실제 DOM으로 만든다.
- [x] 결과적으로, JSX를 실제 DOM으로 변환할 수 있도록 만들었다.
</details>

<details><summary>2) 이벤트 위임</summary>

- [x] 노드를 생성할 때 이벤트를 직접 등록하는게 아니라 이벤트 위임 방식으로 등록해야 한다
- [x] 동적으로 추가된 요소에도 이벤트가 정상적으로 작동해야 한다
- [x] 이벤트 핸들러가 제거되면 더 이상 호출되지 않아야 한다
</details>

### 심화 과제
<details><summary>1) Diff 알고리즘 구현</summary>

- [x] 초기 렌더링이 올바르게 수행되어야 한다
- [x] diff 알고리즘을 통해 변경된 부분만 업데이트해야 한다
- [x] 새로운 요소를 추가하고 불필요한 요소를 제거해야 한다
- [x] 요소의 속성만 변경되었을 때 요소를 재사용해야 한다
- [x] 요소의 타입이 변경되었을 때 새로운 요소를 생성해야 한다
</details>
<details><summary>2) 포스트 추가/좋아요 기능 구현</summary>

- [x] 비사용자는 포스트 작성 폼이 보이지 않는다
- [x] 비사용자는 포스트에 좋아요를 클릭할 경우, 경고 메세지가 발생한다.
- [x] 사용자는 포스트 작성 폼이 보인다.
- [x] 사용자는 포스트를 추가할 수 있다.
- [x] 사용자는 포스트에 좋아요를 클릭할 경우, 좋아요가 토글된다.
</details>

## 과제 셀프회고

### 기술적 성장

- 준일 코치님의 블로그글을 몇년전? 에 리액트 처음 공부하면서 봤었던 많이 참고 했던 글이었는데, 멘토링을 받으면서 알게되어 영광이었습니다.
- React의 기본 개념의 기반을 해당 블로그 글이었고, 현재도 React가 어떻게 동작하냐 면접같은 경우에 물어본다면 해당 글의 내용을 기억 속에서 꺼내서 대답할 것 같습니다.

### 코드 품질

### 학습 효과 분석
- 1주차 때는 구현 먼저 다 해놓고 테스트를 나중에 봤는데, 이번엔 테스트 코드의 순서대로 하나씩 해결을 하니 로직의 이해가 확실히 수월했습니다. 
(마치 게임의 퀘스트를 하는 느낌의 성취감)

### 과제 피드백
- createVNode에서 평탄화가 왜 필요할까? 
- 2중, 3중으로 배열을 엮는 경우가 있을까?  
- 결국 React.Fragment <></> 에 감싸지는 것은 아닌가? 
- 하는 의문이 들었었습니다.

```javascript
const some = [
  { name: "Parent 1", sub: [{ name: "Child 1-1" }, { name: "Child 1-2" }] },
  { name: "Parent 2", sub: [{ name: "Child 2-1" }] },
];

const other = [{ name: "Other 1" }, { name: "Other 2" }];
```

```jsx
/** @jsx createVNode */
import { createVNode } from "./lib"; 

const NestedList = ({ some = [], other = [] }) => {
  return (
    <ul>
      {some.map((so, idx) => (
        <>
          <li key={idx} data-indent={1}>
            {so.name}
          </li>
          <>
            {so.sub?.map((su, jdx) => (
              <li key={`${idx}-${jdx}`} data-indent={2}>
                {su.name}
              </li>
            ))}
          </>
        </>
      ))}
      {other.map((item, idx) => (
        <li key={`other-${idx}`} data-indent={1}>
          {item.name}
        </li>
      ))}
    </ul>
  );
};

```

- createVNode로 표기해보자면 다음과 같을 것입니다.
```javascript
createVNode(
  "ul",
  null,
  [
    // 이게 some.map(...) 결과
    [
      createVNode(Fragment, null, // 첫 번째 Fragment
        createVNode("li", null, so.name),
        createVNode(Fragment, null, // 두 번째 Fragment
          createVNode("li", null, su.name),
          createVNode("li", null, su.name)
        )
      )
    ],
    [ ... ]
  ]
)
```

- type 만 표기해보자면 다음과 같이 최종적으로 ul에 담기게 되다보니(중간에 flat이 없다면..) children = [[[vNode]]] 구조가 되어 예외가 발생하게 됩니다.
```javascript
  createVNode("ul", null, [ [ [ li, [li, li] ] ], ... ])
```
다만, 이러한 상황이 발생한다는 것은 개발자가 코드를 잘 분리하지 못하는 상황이 된 것으로 예상이 됩니다. 
다중 depth 트리 구조의 상품 카테고리를 seqno, parantSeqno로 연결하고, indent로 한번에 보여줘야 하는 상황에서 사용했던 경험이 있는데, 하위 카테고리들을 더 작은 단위의 컴포넌트로 한번 감싸서 관리하는 것이 코드상으로도 좋았을 것 같다는 생각이 들었습니다.


## 리뷰 받고 싶은 내용

- 팀 멘토링과 세션에서 연달아 설명을 듣게 되어 궁금한 점들을 모두 해소하였습니다.
  - 세션때 남는 시간에 여쭤봤던 url PREFIX를 붙히는 방법에 대하여서도 해결하였습니다.  
- 특히나 nomalizeVNode 과정을 글로만 봤던 때와 다르게, 직접 과제를 통해 구현하는 경험이 재미있었습니다.
- 나머진 제가 공부해야 할 사항인 것 같습니다. 😊
