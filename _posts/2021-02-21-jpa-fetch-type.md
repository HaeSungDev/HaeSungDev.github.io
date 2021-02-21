---
title:  "JPA fetch type(EAGER, LAZY) 비교"
search: true
categories: 
  - Jpa
last_modified_at: 2021-02-21T15:37:32
---

Jpa를 사용하면 테이블을 Entity 객체에 매핑해주고 테이블간의 관계를 객체 레퍼런스 관계로 매핑해줍니다. 따라서 개발자가 직접 SQL를 작성해서 매핑하는 작업을 하지 않고 개발을 할 수 있어서 개발 생산성이 올라갑니다.

하지만 동작하는 원리를 잘 이해하지 못하고 사용하면 애플리케이션 성능에 악영향을 끼칠수 있습니다. 이번 포스팅에서는 애플리케이션 성능에 영향을 줄 수 있는 연관관계 fetch type에 대해 알아보도록 하겠습니다.

## 연관관계 Fetch type

Entity간의 연관관계가 존재할 때 객체의 레퍼런스를 이용해서 연관관계가 있는 Entity에 접근할 수 있습니다. 예를들어, 회사라는 Entity와 직원이라는 Entity는 일대다(OneToMany) 관계를 가지고 있을떄 회사에 속해있는 직원 목록을 다음과 같이 가져올 수 있습니다.

```
Company company = companyRepository.findById(1L);
company.getEmployeeList();
```

id가 1인 회사 정보를 DB에서 Company Entity를 로딩한 다음 연관관계를 통해 직원 목록에 접근했습니다. 이 때 Employee Entity 목록도 DB에 저장된 정보이므로 SQL을 통해 가져오게 됩니다.

연관관계에 있는 Entity에 접근시 DB에서 불러오는 방식은 두가지 방식이 있습니다. 첫번째로 즉시(EAGER) 로딩 방식은 Company Entity를 DB에서 조회할 때 JOIN을 이용해서 직원 정보를 "즉시" 영속성 컨텍스트에 로딩합니다. 두번째로 지연(LAZY) 로딩 방식은 우선 Company Entity만 영속성 컨텍스트에 로딩하고 직원 정보는 로딩하지 않고 접근시에 DB에서 로딩합니다.

예제를 통해 즉시 로딩과 지연 로딩을 비교해보도록 하겠습니다.

예제에서 사용할 Entity 코드입니다.

```
// 회사 Entity
@Entity
@Getter
class Company {
  @Id
  @GeneratedValue()
  private Long id;

  @Column(name = "company_name")
  private String companyName;

  @OneToMany
  @JoinColumn(name = "company_id")
  private List<Employee> employeeList;
}

// 직원 Entity
@Entity
@Getter
class Employee {
  @Id
  @GeneratedValue
}

```
### 즉시 로딩

```
@Entity
class Company {
  @Id
  @GeneratedValue()
  private Long id;
}
```

### 지연 로딩


## 참고

자바 ORM 표준 JPA 프로그래밍 책