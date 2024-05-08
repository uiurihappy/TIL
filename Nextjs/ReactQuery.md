## React Query에 대해

이제 Redux 대신에 왜 React-Query를 쓰는가
(내가 알기론 Redux가 전역 상태 관리로 여러 컴포넌트가 공유되는 상태를 관리하기 위한 라이브러리로 알고 있움)

Redux saga, toolkit을 써도 서버 데이터를 불러와 상태 관리 영역에 넣어둘 수는 있다.
그런데 굳이 왜 React-Query를 쓸까?

React-Query의 핵심은 데이터를 가져오는 것이다.
서버에서 데이터를 가져오는 Redux의 핵심은 데이터를 컴포넌트 간에 공유하는 것이다.

벌써 이 둘의 차이는 명확하게, "공유"와 "가져오는 행위"에 초점이 두고 있다.

React-Query는 캐싱이 어마무시하게 좋다고 한다.
요즘엔 트래픽이 나오고 사용자의 요청량이 커지면서 서버에 대한 부담을 확실히 줄여주는 데에 도움이 된다. (또 제가 백엔드라 와닿았어요)

매번 네트워크 통신으로 인한 새로운 데이터를 가져오는 것이 아닌, 캐싱을 사용하여 트래픽을 줄인다. 왜 캐시를 사용하는 것이 중요한 지는 본인이 생각해보고 알아보자.

캐싱을 잘해놓는 다는 것이 서버 측 트래픽을 아끼는 이점도 중요하지만, 사용자 UX 측에서도 빠른 데이터를 fetch할 수 있는 장점이 존재한다.

위 장점을 가지고 있는 것이 React-Query이다.

컴포넌트 간에 공유할 때는 그냥 컨텍스트 API 이런 것도 써도 되기 때문에 Redux보다 Zustand라는 경량화 라이브러리로 조금 더 다루기쉬운 상태관리 라이브러리를 쓰곤 한다.

React-Query의 또 다른 장점은 표준화된 인터페이스가 존재한다.
서버 측 데이터를 가져올 때 필요한 로딩 성공, 실패 등등 인터페이스가 표준화 되어있다.
또한 React-Query는 비동기 상태를 편하게 사용하고 관리하고 싶을때 사용한다

---

## React Query 상태

### Fresh

기본적으로 서버에서 데이터를 들고 오면 fresh이다.
fresh는 이 데이터는 항상 지금 최신 데이터고 굳이 업데이트할 필요가 없기에 계속 사용할 수 있는 상태이다. 즉, 그냥 캐시에 있는 데이터를 사용해도 되는 상태이다.
React-Query의 모든 데이터는 fresh가 아니다라고 기본값 설정이 되어있다.

### Fetching
데이터를 가져올 때,

### Paused
잠깐 데이터를 가져오는 것을 멈출 수 있는 기능이 있는데 멈춘 상태이다.
(ex. 오프라인 상태)

### Stale

기회가 되면 데이터를 새로 가져와라. 라는 뜻이다.

```javascript
# RQProvider.tsx
const [client] = useState(
    new QueryClient({
      defaultOptions: {
        // react-query 전역 설정
        queries: {
        //   stale 된 세 가지 데이터 기준
          refetchOnWindowFocus: false,  // window focus
          retryOnMount: true,  // component 가 unmount 되었다가 다시 mount
          refetchOnReconnect: false,  // network 연결 여부에 따라 데이터를 가져오는 것

          retry: false,
        },
      },
    }),
  );
```

```javascript
export default function PostRecommends() {
  const { data } = useQuery<IPost[]>({
    queryKey: ['posts', 'recommends'],
    queryFn: getPostRecommends,
    // 기본값은 0이다. fresh to stale state가 되는 시간이다. (milliseconds)
    // 즉, 서버 측에서 데이터를 가져온 후 1분뒤 stale 상태가 되고, 이후에 다시 서버에서 데이터를 가져온다.
    staleTime: 60 * 1000,
    // gcTime(가비지 컬렉터 타임) 은 캐시 데이터를 지우는 시간이다. (milliseconds)
    gcTime: 300 * 1000, // default: 5분
  });

  return data?.map((post) => <Post key={post.postId} post={post} />);
}
```

위 코드에서 보다시피 기회가 되면 데이터를 가져오라는 뜻이 1분 뒤 stale 상태가 되었으니 서버에서 데이터를 새로 fetch하는 뜻이다.
컨텐츠 및 기획에 따라 시간을 정해주면 된다.

### Inactive

캐시 여부가 아닌 페이지에서 키가 사용하냐, 안하냐에 따른 상태이다.
inactive 상태가 시작되면 gcTime이 돌아간다.
설정된 시간 사이에는 캐시에서 매번 데이터를 불러오지만, 5분이 지나서 메모리가 정리가 되버리면 캐시가 날라가기 때문에 새로 다시 불러와야 한다.

일반적으로는 gcTime을 staleTime보다 더 길게해야 한다. 
(gcTime > staleTime)
안그러면 inactive 상태를 사용하는 의미가 없고 staleTime이 캐시를 얼마동안 오래 간직할까에 대한 의도이기에 gcTime, inactive 일 때 메모리 정리하는 gcTime은 staleTime보다도 길어야 정확한 의도가 된다.

• gcTime 의 기본값 5분, staleTime 기본값 0초를 가정
1. A 라는 queryKey를 가진 A 쿼리 인스턴스가 mount 됨
2. 네트워크에서 데이터 fetch하고, 불러온 데이터는 A라는 queryKey로 캐싱 함
3. 이 데이터는 fresh 상태에서 staleTime(기본값 8) 이후 stale 상태로 변경됨
4. A 쿼리 인스턴스가 unmount 됨
5. 캐시는 gcTime(기본값 5분) 만큼 유지되다가 가비지 콜렉터(GC)로 수집됨
6. 만일, gcTime 지나기 전이고, A 쿼리 인스턴스 fresh한 상태라면 새롭게 mount되면 캐시 데이터를 보여준다.


## React-Query 액션 (refetch, invalidate, reset)

### 1. refetch
무조건 새로운 데이터를 호출하는 액션.

### 2. invalidate
새로운 데이터를 호출하는 액션이지만, React-Query 키를 더이상 사용하지 않기에, 새로운 데이터를 호출하는 것이다.
여기서 observer는 지금 현재 페이지에서 이 데이터를 사용하고 있는 데이터를 가리킨다.
다른 페이지로 이동을 하면 observer가 0이 되어버리고 이때, invalidate를 하게 되면 바로 가져오지 않고 observer가 다시 1이 되면 그때 가져온다.
refetch보다는 좀 더 효율적인 액션이다. invalidate는 결국 무조건 가져오는게 아닌 inactive일때는 안가져오고, 페이지에서 데이터를 쓰고 있을때만 가져오는 것이다.

### 3. reset
initialData로 초기 데이터가 존재한다면, 초기 데이터로 세팅하고, 없는 경우는 데이터를 새로 가져온다.

## React Query에서 쿼리 결과를 캐시에서 지우는 방법은 무엇인가?

React Query에서는 QueryClient 인스턴스의 removeQueries 메소드를 사용하여 캐시에서 쿼리 결과를 지울 수 있다. 이 메소드는 쿼리 키를 인자로 받아 해당 쿼리의 결과를 캐시에서 제거한다.

다음은 removeQueries 메소드를 사용하는 예제이다.

```javascript
const queryClient = new QueryClient();

// 쿼리 실행
queryClient.prefetchQuery('posts', fetchPosts);

// 쿼리 결과 제거
queryClient.removeQueries('posts');
```

위의 코드에서 fetchPosts는 데이터를 가져오는 함수이며, 'posts'는 쿼리 키이다. removeQueries 메소드를 호출하면 'posts' 쿼리의 결과가 캐시에서 제거된다.

또한, removeQueries 메소드는 옵션 객체를 두 번째 인자로 받을 수 있다고 한다. 이 옵션 객체를 통해 쿼리 제거의 동작을 더 세밀하게 제어할 수 있다. 예를 들어, exact 옵션을 false로 설정하면 부분적으로 일치하는 쿼리 키를 가진 모든 쿼리의 결과가 제거된다.