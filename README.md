### supabase 소셜 로그인

- authentication -> sign in/providers 들어가면 소셜 쭉 뜸
- 깃허브 예로 깃허브 누르면 github enabled 체크 하고 client id, client secret 채우기
- callback url 복사해서 github setting -> developer ssettings 에서 outh 만든거 들어가서 callback url 채우고
- 거기서 받는 client id랑 secret key 받아와서 supabase에 채우고 save, update하기
- docs 에 있는 코드로 소셜 로그인 구현

#### 소셜 로그인 확인법

- supabase에 authentication에 user 들어가보면 값이 있으면 로그인 성공, 없으면 실패

### supabase 관리(소셜 로그인 테이블로 넘어가게 하는 것)

- 데이터베이스 '테이블(table)'로 관리해야 함
- 로그인 -> 로그인에 대한 정보 -> 내가 관리하는 데이터베이스의 테이블로 넣어야 함
- 소셜 로그인을 하면 자동으로 profiles 테이블에 로그인한 사용자의 정보가 들어가게 할 것
- 데이터베이스 누르면 만든 거 확인 가능(테이블을)
- 테이블 만든 후, database -> functions -> new function create 만들어주기
- sql editor에 trigger를 만들어줘야 함, 여기서 내가 만든 function이랑 이름이 일치해야 함, 그리고 run으로 실행
  - database에 있는 triggers에서 trigger 잘 만들어졌는지 확인 가능
- 권한 부여 : sql editor에서 코드 써서 똑같이 run
- 정책 만들기(허가해주는 걸 만드는 것) : database -> policies -> create policy -> 이름 지어주기 -> insert 고르고 코드 밑에 true 써주기

  - 이렇게 하면 누구나 권한을 부여해주는 것(접근 가능하게)

- 이렇게 하면 authentication user에도 들어가고, table editor에도 들어감

#### foreign key

- schema : auth
- reference : users
- id끼리 연결, 나머지는 no action

#### error 확인

- logs -> auth에서 확인 가능

### profile setting

- 초기에 받지 못한 정보를 받기 위한 기능 만드는 것
- 이미 있는 정보들은 supabase에 담겨 있는 것들 가져와서 띄우기도 추가

- 테이블에 필요한 컬럼 추가해주기
- api docs -> guides에서 getClaims 검색하면 사용자 정보 조회가능
- 거기서 넘어오는 데이터가 어떻게 넘어오는지 확인할 수 있는데, 그 데이터를 받기 위해서는 타입 지정해야 하니까 맞게 타입 정해주기
- JwtPayload : 완성된 타입, 이걸 활용해서 내가 원하는 데이터만 타입 지정해줄 수 있게 함 -> JwtPayload & {내가 원하는 타입 지정} | null, 이런 식으로
- user_metadata 가져올 때는 개발자도구 application에서 local storage에서 넘어온 키 보고 뭐 넘어오는지 copy 해서 손쉽게 타입 지정

- profilesetup에 정보를 얻을 수 있게 useEffect 작성, 여기서 data를 받을 때 as Claims 쓰는 이유는 user_metadata를 제대로 못 써서 Claims를 써줌으로서 타입을 확정? 해주는 개념인 듯

  - 다른 정보 : d.ts 파일에서 글로벌하게 인식 시켜주기 위해서는 그 파일 안에 import가 없어야 함, 그래서 as Claims할 때 불러와서 지정해주는 것

- api docs -> introduction에 우리가 만든 data type을 다운로드 할 수 있음, 다운 받은 거 type 폴더에 d.ts파일로 만들어줌
- 그걸 utils에 있는 supabase 파일에 있는 createClient 옆에 <Database> 이런 형식으로 넣어주면 계속 지정이 됨, supabase 쓰는 일에 대해서는
- 데이터 조회는 api docs -> profiles를 통해 조회하는 방법이 다 나와있음
- 거기서 맞는 데이터 아까 만든 useEffect에 넣어서 하는 것, 근데 권한을 지정해줘야 함
- database -> policies 에서 새로운 policy 만들어주기(모두 허용할거면 마지막에 true 작성)

- 이렇게 하면 넘어오는 데이터는 모든 데이터를 조회하는 것, 그래서 api docs->profiles에서 필터링하는 방법이 있음

- 필터링 코드

  - eq : 컬럼값이 같은지 검사
  - gt : 컬럼값이 특정 값보다 큰지 검사
  - lt : 컬럼값이 특정 값보다 작은지 검사
  - gte : 컬럼값이 특정 값보다 크거나 같은지 검사
  - lte : 컬럼값이 특정 값보다 작거나 같은지 검사
  - like : 특정 값이 포함되어 있는지 검사(대소문자 구분)
  - ilike : 특정 값이 포함되어 있는지 검사(대소문자 구분X)
  - is : 컬럼의 값이 'null'인지 검사
  - in : 배열 안의 있는 값이 하나라도 컬럼 값과 매칭되는지 검사
  - neq : 특정 값과 같지 않은지 검사
  - 여기서는 eq를 써서 같은지 검사만 할 것

- 그럼 어떻게 검사를 할것인가? 아까 정했던 claims와 매칭을 해줄 것임
- claims와 profiles(받았던 데이터) 데이터를 매칭해보면, id에 있는 값이 sub에 있는 걸 알 수 있음, 그래서 둘이 같은지 매칭하는 것
- select 뒤에 .eq("id", claims?.sub || ""); 이런식으로 붙여서 둘이 같은지 확인함
- 마지막에 .single() 를 붙여주면 내 정보가 객체 형태로 넘어옴
- 이제 내가 원하는 useState에 넣어서 데이터가 화면에 보이게 세팅해주면 끝~

#### update policy

- update를 할 때도 policy를 만들어줘야 함(authentication -> policies)
- update로 해주고, roles에 authentication 넣고 두 빈칸에 auth.uid() = id 넣기
- auth.uid는 authentication에 있는 uid를 가져오는 것, 이건 테이블에 있는 uuid(내가 지정한 것) 와 같은 것
- 사용자가 얻어오는 유효 id 값과 테이블에 있는 유효 id 값과 같을 때만 update가 가능하게 해준다는 것

### 입력받은 bio(또는 다른 데이터) 테이블로 받아오는 법

- api docs -> profiles에 있는 updata rows 받아와서 붙여놓기, 붙여놓는 곳은 buttons, 제출할 때 눌려지는 onClick 함수에다 붙여놓는 것
- 이때 .eq를 할 때 claims는 useEffect에만 있는 값임, handle에는 없음(근데 .eq는 무조건 해야 함, 아니면 다른 모든 값들이 바뀌게 됨)
- 어떻게 가져오냐? useState를 새로 만들어서 값을 거기에 넣음(값을 조회했을 때)
- claims.sub은 eq할 때 claims!.sub으로 null 아님 연산자 적용 또는 위에 타입가드를 써주면 됨
- 이 내용을 try, catch로 묶어주고 data가 있다면 홈으로 navigate

#### details

- 이미 bio 값이 있다면 바로 넘어갈 수 있게 이런 걸 useEffect try 문에 넣는 것도 디테일적인 방법
  ```
  if (profiles.bio) {
          navigate("/blog");
        }
  ```

### zustand 써서 로그인 정보 전역으로 쓰기

- authStore로 zustand 만들기
- profile(profile 테이블 데이터 담는 곳) 타입 지정해줄 때, 그냥 다 d.ts 새로 만들어서 적어도 되지만, 그 속에서 아까 만든 database.d.ts에서 가져올 수 있음(웬만하면 타입 다 있기 때문)
- onAuthStateChange : api docs guides 에서 검색하면 됨
- 만든 zustand를 main에 App 위에 새로 껴놓으면 됨
