
### 공부 루트 제안 (실전형)

1. **값 카테고리 & auto 추론만 따로 정리**

   * 짧은 코드 20개 정도 만들어서:

     * `auto`, `auto&`, `auto&&`
     * `const`, `&&` + lvalue/rvalue
     * `std::move`, `std::forward`
   * 타입을 맞춰보는 훈련을 해보면 Q1, Q5 계열이 확실히 정리될 거야.

2. **rule-of-zero/three/five 정리**

   * “소멸자/복사/이동을 내가 하나라도 정의하면, 나머지가 어떻게 바뀌는가”만
     표로 정리해서 한 번 외워두면 Q3 계열 걱정이 거의 사라짐.

3. **move + optional/variant/컨테이너 실험**

   * `optional<string>`에서 `value()`를 move해 보고
   * 그 뒤에 `has_value()`, 내부 값 상태를 로그로 확인
   * `vector<unique_ptr<T>>` 이런 것도 직접 써보면서 **“move-only 타입과 컨테이너”** 감각 익히기

4. **pmr/allocator는 “필요할 때 다시 오기”**

   * 지금은 개념만 알고 있으면 충분해 보여
   * 진짜 오디오 실시간 경로에서 할당 병목이 보이면, 그때 `monotonic_buffer_resource` 실험을 별도 미니 프로젝트로 하는 게 효율적.


