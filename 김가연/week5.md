## 변경 감지와 병합

변경 감지와 병합(merge)는 모두 데이터를 수정할 때 사용하는 방식이다. 둘의 차이점은 무엇일까? 지금부터 살펴보자.

### 병합 (merge)

병합, `em.merge()` 메서드는 Detached(준영속) 상태의 Entity를 다시 영속화 하는데 사용한다. 여기서 준영속 엔티티란 영속성 컨텍스트가 더는 관리하지 않는 엔티티를 뜻한다. 아래 코드를 살펴보자.

```java
public String updateItem(@ModelAttribute("form") BookForm form) {
     Book book = new Book();
     book.setId(form.getId());
     book.setName(form.getName());
     book.setPrice(form.getPrice());
     book.setStockQuantity(form.getStockQuantity());
     book.setAuthor(form.getAuthor());
     book.setIsbn(form.getIsbn());
     itemService.saveItem(book);
     return "redirect:/items";
 }
```

위의 코드에서 Book 객체는 준영속 상태이다. `setId()` 메서드에서 확인할 수 있듯 Book 객체는 DB에 한번 저장되어 식별자가 존재하는 객체이다. 하지만 우리는 `new` 연산자를 통해 새로운 객체를 생성하였기 때문에 현재 book 객체는 영속성 컨텍스트가 관리하지 않는다. 즉, 정리하면 **준영속 엔티티**의 핵심은 **‘식별자를 기준으로 영속상태가 되어 DB에 저장된적이 있는가’**이다. 이러한 준영속 엔티티는 JPA가 관리를 하지 않기 때문에 객체를 수정을 해도 DB에 Update가 일어나지 않는다.

```java
@Transactional
void update(Item itemParam) { //itemParam : 파리미터로 넘어온 준영속 상태의 엔티티
     Item mergeItem = em.merge(itemParam);
     // itemParam은 여전히 준영속, mergeItem은 영속 상태가 된다.
 }
```

이러한 준영속 엔티티를 수정하기 위한 방법 중 하나로 병합이 존재한다. `em.merge()` 메서드를 호출함으로써 준영속 상태의 엔티티를 영속 상태로 변경한 후 데이터를 수정하는 것이다.

![](https://velog.velcdn.com/images/yeoni_/post/9c6668ce-91a1-4eb0-b69d-805e58257ac7/image.png)


먼저 `merge()`가 실행되면 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다. 이때, 1차 캐시에 엔티티가 존재하지 않으면 DB에서 엔티티를 조회하고 1차 캐시에 저장한다. 이렇게 조회한 영속 엔티티의 값을 member 엔티티(준영속 상태였던 엔티티)의 값들로 모두 교체한다. HTTP 메서드 중 PUT과 비슷하게 동작한다고 생각하면 된다. 그 후 트랜잭션 커밋 시점에 변경 감지 기능이 동작하여 DB에 update 쿼리가 실행된다. 

하지만 이러한 병합 방식은 원하는 속상만 선택해서 변경할 수 없다. 즉, 모든 필드들이 통째로 교체되기 때문에 병합 시 값이 존재하지 않는다면 null 값으로 업데이트 되는 위험이 존재한다. 아까의 예시 코드에서 만약 `book.setAuthor(form.getAuthor());` 를 작성하지 않았다면 Author 속성이 null값으로 변경되어 DB에 저장된다는 뜻이다. 그렇기 때문에 병합보다는 변경 감지를 이용하여 데이터를 수정하는 것을 권장한다.

### 변경 감지 (dirty checking)

앤티티를 변경할 때 가장 좋은 방법은 변경 감지를 사용하는 것이다. 

```java
@Transactional
public void updateItem(Long id, String name, int price, int stockQuantity) {
    Item item = itemRepository.findOne(id);
    item.setName(name);
    item.setPrice(price);
    item.setStockQuantity(stockQuantity);
}
```

위의 코드처럼 `findOne()` 메서드를 통해 Item 객체를 영속화한 후, 데이터를 수정하면 트랜잭션 커밋 시점에 변경 감지가 실행되어 자동으로 update 쿼리가 실행된다.