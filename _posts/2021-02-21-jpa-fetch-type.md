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

연관관계에 있는 Entity에 접근시 DB에서 불러오는 방식은 두가지 방식이 있습니다. 

* 첫번째로 즉시(EAGER) 로딩 방식은 Company Entity를 DB에서 조회할 때 JOIN을 이용해서 직원 정보를 "즉시" 영속성 컨텍스트에 로딩합니다.

* 두번째로 지연(LAZY) 로딩 방식은 우선 Company Entity만 영속성 컨텍스트에 로딩하고 직원 정보는 로딩하지 않고 추후 접근시에 DB에서 로딩합니다.

예제를 통해 즉시 로딩과 지연 로딩을 비교해보도록 하겠습니다.

## 즉시 로딩

예제에서는 Company Entity와 Employee Entity 사이에 일대다 연관관계를 설정하고 발생하는 쿼리를 확인해보겠습니다.

**Entity 설정**
```
// 회사 Entity
@Entity
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Company {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "company_id")
    private Long id;

    @Column(name = "company_name")
    private String companyName;

    @OneToMany(mappedBy = "company", fetch = FetchType.EAGER, cascade = CascadeType.ALL)
    private List<Employee> employeeList;

    public void addEmployee(Employee employee) {
        if (employeeList == null) {
            employeeList = new ArrayList<>();
        }
        employeeList.add(employee);
    }
}

// 직원 Entity
@Entity
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "employee_id")
    private Long id;

    @Column(name = "employee_name")
    private String employeeName;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "company_id", nullable = false)
    private Company company;
}
```

**즉시 로딩 테스트**
```
@SpringBootTest
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public class FetchTest {
    @Autowired
    CompanyRepository companyRepository;

    Long companyId;

    @BeforeAll
    public void testData() {
        Company company = Company
                .builder()
                .companyName("NHN")
                .build();

        company.addEmployee(Employee.builder().employeeName("이해성1").company(company).build());
        company.addEmployee(Employee.builder().employeeName("이해성2").company(company).build());
        company.addEmployee(Employee.builder().employeeName("이해성3").company(company).build());
        company.addEmployee(Employee.builder().employeeName("이해성4").company(company).build());
        company.addEmployee(Employee.builder().employeeName("이해성5").company(company).build());

        companyRepository.save(company);

        companyId = company.getId();
    }

    @Test
    @Transactional(readOnly = true)
    public void eagerFetchTest() {
        Company company = companyRepository.findById(companyId).get();

        assertEquals(5, company.getEmployeeList().size());
    }
}
```

즉시 로딩 방식으로 위 테스트를 진행하면 다음과 같은 쿼리가 발생합니다. 보시다시피 join으로 처음부터 직원 정보를 함께 가져오는 것을 확인할 수 있습니다.

위 테스트 코드에서 company.getEmployeeList().size() 를 빼고 다시 테스트를 진행해도 동일한 쿼리로 데이터를 가져오는 것을 확인할 수 있습니다.

```
Hibernate: 
    select
        company0_.company_id as company_1_0_0_,
        company0_.company_name as company_2_0_0_,
        employeeli1_.company_id as company_3_1_1_,
        employeeli1_.employee_id as employee1_1_1_,
        employeeli1_.employee_id as employee1_1_2_,
        employeeli1_.company_id as company_3_1_2_,
        employeeli1_.employee_name as employee2_1_2_ 
    from
        company company0_ 
    left outer join
        employee employeeli1_ 
            on company0_.company_id=employeeli1_.company_id 
    where
        company0_.company_id=?
```

## 지연 로딩

즉시 로딩에서 사용했던 예제를 동일하게 사용하고 FetchType.EAGER만 FetchType.LAZY로 변경해서 테스트를 진행했습니다.

지연 로딩으로 설정한 연관관계에서는 다음과 같이 쿼리로 데이터를 가져옵니다.

```
Hibernate: 
    select
        company0_.company_id as company_1_0_0_,
        company0_.company_name as company_2_0_0_ 
    from
        company company0_ 
    where
        company0_.company_id=?
Hibernate: 
    select
        employeeli0_.company_id as company_3_1_0_,
        employeeli0_.employee_id as employee1_1_0_,
        employeeli0_.employee_id as employee1_1_1_,
        employeeli0_.company_id as company_3_1_1_,
        employeeli0_.employee_name as employee2_1_1_ 
    from
        employee employeeli0_ 
    where
        employeeli0_.company_id=?
```

회사 정보를 가져올 때는 직원 정보를 가져오지 않고 나중에 company.getEmployeeList().size()를 호출할 때 아래 쿼리로 직원 정보를 불러옵니다.

company.getEmployeeList().size()가 포함된 줄을 삭제하고 다시 테스트를 실행해보면 아래 쿼리는 호출되지 않는 것을 확인할 수 있습니다.

getEmployeeList는 단순히 리스트를 반환하는 메소드인데 어떻게 지연 쿼리를 호출할 수 있을까요? Jpa 구현체인 Hibernate에서는 이를 Proxy 패턴을 이용해서 구현했습니다.

간단하게 설명하면 List<Employee> employeeList를 상속한 Proxy 객체를 만들어서 모든 멤버 메소드를 재정의해서 접근시에 영속성 컨텍스트에 해당 Entity가 존재하지 않으면 DB에 접근하는 코드를 추가합니다. 따라서 위 예제에서는 size 메소드를 호출할 때 DB에서 데이터를 가져옵니다.

## 즉시로딩 vs 지연로딩

즉시로딩과 지연로딩 중 어떤 것을 사용할지는 애플리케이션 구조에 따라 최적화를 해야합니다. 

항상 여러 Entity가 한번에 로딩돼서 사용되는 것들은 즉시 로딩, 그렇지 않고 일부분의 Entity만 사용되는 것은 지연 로딩을 사용해서 최적화하면 됩니다.

하지만 즉시로딩의 경우 다음과 같은 상황에 문제가 발생할 가능성이 높습니다.

앞서 본 예제에서 회사의 목록을 가져오는 상황이 있을 때 다음과 같은 코드를 사용할 수 있습니다.
```
companyRepository.findAll(pageRequest);
```

이런 상황에서 즉시 로딩을 하게 되면 회사의 개수만큼 직원들의 정보를 가져오는 N+1 쿼리 문제가 발생합니다.

따라서 Jpa 구현체인 Hibernate 공식 문서에서는 성능 최적화 가이드로 다음과 같이 제시하고 있습니다.

> 즉시 로딩은 대부분의 상황에서 좋지 않은 선택이므로 지연 로딩을 기본으로 하고 연관관계가 있는 Entity에 접근할 때는 JPQL의 JOIN FETCH 지시자를 이용해서 로딩하는 것이 좋습니다.

결론적으로 FetchType은 특별한 경우가 아니면 즉시 로딩 방식은 사용하지 않고 지연 로딩 + JOIN FETCH 지시자를 사용하는 것이 좋을 것 같습니다.

## 참고

책 자바 ORM 표준 JPA 프로그래밍(저자: 김영한)

[Fetching Best Practice](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#best-practices-fetching-associations)