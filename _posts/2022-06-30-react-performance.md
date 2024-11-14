---
title: "브라우저에 데이터 저장하기"
author:
  name: yeong30
  link: https://github.com/yeong30
date: 2022-06-30 20:00:00 +0900
categories: [Brower]
tags: [Brower]
---

## 성능 측정하기
React에서 성능 최적화는 가장 중요한 요소 중 하나이다.
토이프로젝트를 위해 최적화를 하는 방법을 찾아보다 React Developer Tools에 있는 Profiler라는 기능을 알게되었다, 

## React Developer Tools with Profiler

React Developer Tools은 오픈소스 React JavaScript 라이브러리용 Chrome DevTools 확장 프로그램으로 React의 컴포넌트 계층구조등을 검사할 수 있다.
React Developer Tools은 구성요소, 프로파일러라는 두개의 탭을 갖는데 구성요소는 돔트리 처럼 실제 컴포넌트의 트리를 볼 수 있다. 
프로파일러탭에서는 성능정보를 기록하고 조회할 수 있다.

이중 프로파일러 기능에 대해 기록하고자 한다.

## 성능을 개선하자
우선 성능을 개선하기 위해서는 성능을 측정, 비교 후 더 나은 방식을 선택해야한다.
진행중인 프로젝트에서 검색을 하고 무한 스크롤로 데이터를 가져오는 컴포넌트를 대상으로 개선을 진행하고자한다.
사용방법은 [여기](https://moood.dev/reactjs/performance-profiling-your-react-app/)를 참고한다.

### Case1
검색form에서 검색을 하면 redux store에 데이터가 저장되고 List 컴포넌트에서 해당 데이터를 가져와 출력하는 컴포넌트이다.
코드는 다음과 같다.


in SearchList
```
const { plants, loading, totalCount, error } =seSelector((state) => ({
    ...state.plants,
  }));
 ...
 let content = <PlantList plants={plants} />;
if (loading) content = <LoadingSpinner />;
if (error) return <PlantList error={error} />;

return (
    <Fragment>
    	<SearchForm/>
          ...
        {content}
    </Fragment>
  );
  
```

Profiler를 실행시켜 검색을 진행했을때의 측정 결과이다. 
18번의 commit이 발생했으며 그 중 가장 오래걸린 commit은 아래 사진과 같다.
![](https://velog.velcdn.com/images/pro-yeong/post/caa82fd5-58b2-415e-89b1-584ae5089316/image.png)

plantList가 렌더링되는데 0.4ms가 걸렸으며 그 이유는 부모의 렌더링 때문이라고 한다. 즉 props변경이나 state값의 변경이 아닌 부모 컴포넌트의 변경때문이다.  


부모가 리렌더링 되더라도 자식의 상태의 변화가 없다면 리렌더링 하지않도록 (정확히는 저장해둔 값을 사용하는) 하기 위해 PlantList에 React.memo를 적용하였다.

![](https://velog.velcdn.com/images/pro-yeong/post/cf56ad5e-e11f-42ef-aec6-2f1c0d3c593e/image.png)

그 결과, 같은 commit에서 가장오래걸렸던 PlantList는 렌더링 되지않았으며 그 하위 컴포넌트도 모두 렌더링되지않아 렌더링 시간이 약 5ms 감소하였다. 
전체적인 흐름에서 PlantList의 렌더링 횟수도 9번에서 3번으로 감소하였다. 


### Case2

DevTools은 추가적으로 Highlight updates when components render 기능을 제공한다. 화면상에 파란색 ~ 주황색의 박스로 렌더링 여부를 표기해주는 기능이다. 렌더링 1번은 파랑색 연속적으로 발생하면 주황색으로 표기된다.

측정 대상으로는 검색 form을 선택하였다.
1개의 input, 2개의 selectBox가 있는 검색 form 이다. 

검색 form의 input에 단어를 입력하다보면 사용하지도 않는 selectBox가 계속 주황색으로 깜박거리는것을 볼 수 있었다.

![](https://velog.velcdn.com/images/pro-yeong/post/0589b5c9-d630-4d29-b006-dbe9c4960c16/image.png)

Input에의해 form컴포넌트의 상태가 계속변하면서 하위 요소인 selectBox도 계속 렌더링이 발생하게 되는것이다.

실제로 측정해보면 SelectBox가 여러번 리렌더링 되는것을 볼 수 있다.
![](https://velog.velcdn.com/images/pro-yeong/post/58119a35-f81d-45a5-8783-b3728be466ec/image.png)

이를 위해 SelectBox를 React.memo로 묶고 속성으로 지정된 함수를 useCallback으로 감싸준다.


![](https://velog.velcdn.com/images/pro-yeong/post/b478404c-07f3-4f2d-a8fa-3e07ae81b098/image.png)
그 결과 SelectBox는 아예 렌더링이 발생하지 않을 수 있다.




## 정리하기
여러가지 케이스 적용을 통해 생각보다 최적화가 중요함을 느꼈다. 
단, useCallback, React.memo와 같은 최적화가 항상 최적의 결과를 낳는것은 아님을 항상 기억하자.