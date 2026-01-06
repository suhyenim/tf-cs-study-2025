# 📌 키(Key) 개념

<details> 
<summary>참고: 아래 질문들은 사실 한 줄로 이어져 있음</summary>

---

Row를 유일하게 식별해야 한다.        
→ [DB-001] Key (기본키, 후보키, 슈퍼키 등등...) 에 대해 설명해 주세요.          
→ [DB-002] 기본키는 수정이 가능한가요?          
→ [Db-003] 사실 MySQL의 경우, 기본키를 설정하지 않아도 테이블이 만들어집니다. 어떻게 이게 가능한 걸까요?        
→ [DB-004] 외래키 값은 NULL이 들어올 수 있나요?        
→ [DB-005] 어떤 칼럼의 정의에 UNIQUE 키워드가 붙는다고 가정해 봅시다. 이 칼럼을 활용한 쿼리의 성능은 그렇지 않은 것과 비교해서 어떻게 다를까요?   

Row를 유일하게 식별해야 한다.    
→ 그 식별자를 어떻게 고르고(키)     
→ 그 식별자가 바뀌면 왜 위험한지(PK 수정)             
→ PK가 없으면 DB는 어떻게든 식별자를 만들어야 하고(MySQL/InnoDB)          
→ 관계가 생기면 그 식별자를 다른 테이블이 참조한다(FK, NULL)      
→ 유일성 제약은 결국 인덱스로 이어져 성능 특성이 바뀐다(UNIQUE 성능)       

</details>

### [DB-001] Key (기본키, 후보키, 슈퍼키 등등...) 에 대해 설명해 주세요.

Key는 데이터베이스에서 "수많은 데이터 중 내가 원하는 바로 그 행(Row)을 어떻게 찾을 것인가?"라는 고민에서 출발   
이러한 Key에는 기본키, 후보키, 슈퍼키 등이 있는데, 이들은 `슈퍼키 ⊃ 후보키 ⊃ 기본키`의 포함관계를 가짐

- **슈퍼키(Super Key): 유일성만 만족하는 키** <- 가능한 조합이 많아서 실용적이지 않음
  - ex) (학번 + 이름), (학번 + 이름 + 주민번호) 등 
- **후보키(Candidate Key): 유일성과 최소성을 모두 만족하는 키**    
  - ex) (학번), (주민번호)
- **기본키(PK, Primary Key): 후보키 중 대표키(NOT NULL, UNIQUE)로, 테이블의 식별자**   
  - ex) (학번) 
- **대체키: 후보키 중 기본키로 선택받지 못한 나머지 키들**   
  - ex) (주민번호) 
- **외래키: 다른 테이블의 기본키 또는 UNIQUE 키를 참조하여 테이블 간의 연관 관계를 맺어주는 키**
  - ex) (수강신청 테이블의 학번) -> 학생 테이블의 학번을 참조하여 '누가' 신청했는지 연결함

### [DB-002] 기본키는 수정이 가능한가요?

기본키는 SQL 문법상 수정이 가능하다.    
하지만 PK는 다른 테이블의 외래키가 참조하는 기준이기 때문에, PK를 변경하면 이를 참조하는 모든 FK 값도 함께 변경되어야 한다.    
이를 자동으로 처리하기 위해 ON UPDATE CASCADE를 사용할 수 있지만, 이 경우 연관된 모든 테이블의 대량 UPDATE가 발생하여    
대기 또는 데드락, 인덱스 재정렬, 성능 저하 같은 심각한 부하를 유발할 수 있다.    
따라서 PK는 논리적으로 가능하더라도, 설계상 변경되지 않는 값으로 선택하는 것이 원칙이다.    
- 락(Lock): PK 변경으로 인해 해당 사용자의 주문·리뷰 행 수천 개가 동시에 잠겨 다른 트랜잭션이 대기 상태에 빠진다.
- 인덱스 재정렬: FK 값이 바뀌면서 orders.user_id와 reviews.user_id 인덱스에서 수많은 B+Tree 노드가 delete+insert로 재배치된다.
- 성능 저하: 수백만 건의 연쇄 UPDATE와 인덱스 수정으로 디스크 I/O와 CPU 사용량이 급증해 서비스 응답이 느려진다.

### [DB-003] 사실 MySQL의 경우, 기본키를 설정하지 않아도 테이블이 만들어집니다. 어떻게 이게 가능한 걸까요?

InnoDB가 **Clustered Index 구조**를 유지하기 위해, 어떤 방식으로든 PK 역할을 할 키(정렬키)를 스스로 정하기 때문에 가능하다.

1. **UNIQUE NOT NULL 인덱스 선택**: 테이블 내에 정의된 인덱스 중 중복이 없고(UNIQUE) 비어 있지 않은(NOT NULL) 첫 번째 인덱스를 찾아 클러스터 키로 사용한다.
2. **Internal Row ID 생성**: 만약 PK도 없고 & UNIQUE NOT NULL 인덱스도 없으면, InnoDB는 내부적으로 6바이트 크기의 Hidden Row ID를 생성하여 이를 기반으로 Clustered Index를 구축한다.
    - Hidden Row ID는 테이블 내부에서만 쓰이고, 자동 증가하며, 사용자에게는 보이지 않지만 / 외부에서 참조 불가, FK로 사용 불가, JOIN 기준으로 사용 불가의 단점을 가진다.

<details>
<summary>Internal Row ID 적용 예시</summary>

1. PK가 없을 때, InnoDB가 만드는 테이블

    | _rowid | name | age |
    | ------ | ---- | --- |
    | 1      | 서현   | 22  |
    | 2      | 민수   | 25  |
    | 3      | 지민   | 21  |

2. _rowid 값이 디스크에서 Row의 물리적 순서를 결정
    ```
                [ _rowid 1 | 2 | 3 ]
                /        |        \
        [_rowid=1]   [_rowid=2]   [_rowid=3]
            ↓             ↓             ↓
      (수현,22)      (민수,25)      (지민,21)
    ```

</details>

### [DB-004] 외래키 값은 NULL이 들어올 수 있나요?

외래키 컬럼이 NULL을 허용한다면, FK 값은 NULL이 될 수 있다.    
이는 아직 부모 테이블과 연결되지 않았음을 의미한다.    
외래키 제약은 “반드시 연결돼야 한다”가 아니라    
“NULL이 아닌 값이라면 반드시 부모 테이블의 유효한 키(PK 또는 UNIQUE)여야 한다”를 보장한다.    

### [DB-005] 어떤 칼럼의 정의에 UNIQUE 키워드가 붙는다고 가정해 봅시다. 이 칼럼을 활용한 쿼리의 성능은 그렇지 않은 것과 비교해서 어떻게 다를까요?

> MySQL 기준 + 에시를 통해 설명 + 결론

1. MySQL InnoDB 원칙들
  - InnoDB에서 Clustered Index는 PRIMARY KEY가 있으면 그것으로 만들어지고,
  - 없으면 NOT NULL + UNIQUE 인덱스 중 하나로 만들어진다.
  - 그 외 모든 인덱스(UNIQUE 포함)는 Secondary Index다.

2. email 칼럼에 UNIQUE 키워드를 붙인다고 가정
    ```sql
    users(
      id BIGINT PRIMARY KEY,
      email VARCHAR(50) UNIQUE,
      name  VARCHAR(20)
    )
    ```
    => email 칼럼으로 행(Row) 식별 가능

3. 디스크에 저장되는 실제 데이터 모습
    ```
    [Clustered Index (key = id)] <- InnoDB에서 테이블은 PK B+Tree로 저장됨

    Page A:                        Page B:
    +----------------+            +----------------+
    | id=1 | Alice   |            | id=8 | Carol   |
    | id=3 | Bob     |            | id=10| Dave    |
    | id=6 | Eve     |            | id=15| Frank   |
    +----------------+            +----------------+

    [Secondary Index (key = email)] <- email UNIQUE 때문에 만들어진 인덱스 (이 안에는 row가 없고, PK(id)만 있음)

    Page X:
    +-----------------------------+
    | alice@test.com → 1          |
    | bob@test.com   → 3          |
    | eve@test.com   → 6          |
    +-----------------------------+

    Page Y:
    +-----------------------------+
    | carol@test.com → 8          |
    | dave@test.com  → 10         |
    | frank@test.com → 15         |
    +-----------------------------+

    ```
4. 조회할 때(SELECT): Full Scan보다는 압도적으로 빠르지만, PK(Clustered Index)보다는 항상 한 단계 느리다.
    1. email → PK → row
        1. 사용할 SQL문: `SELECT * FROM users WHERE email = 'bob@test.com';`
        2. DB가 하는 일 1 - Secondary index에서 찾기: `'bob@test.com' → id = 3`
        3. DB가 하는 일 2 - Clustered index에서 찾기: `id = 3 → Page A → Bob row`

5. 추가할 때(INSERT/WRITE): PK 트리와 UNIQUE 트리를 모두 갱신해야 하고, UNIQUE 제약을 지키기 위한 락이 추가로 발생한다
    1. row → PK → UNIQUE → lock
        1. 사용할 SQL문: `INSERT INTO users (id, email, name);` `VALUES (100, 'alice@test.com', 'Alice');`
        2. DB가 하는 일 1 - PK (Clustered Index)에 삽입: B+Tree(id)에서 id=100이 들어갈 leaf page 찾기 -> 페이지에 삽입 -> 필요하면 page split
        3. DB가 하는 일 2 - UNIQUE(email) Secondary Index에 삽입: B+Tree(email)에서 'alice@test.com'이 있는지 먼저 검색 (중복 검사) -> 없으면 삽입 -> 필요하면 page split
        4. DB가 하는 일 3 - 잠금: 'alice@test.com'이 들어갈 **UNIQUE 인덱스의 레코드(또는 해당 gap)**에 락을 걸어, 다른 트랜잭션이 같은 email을 동시에 넣지 못하게 막음

6. 예상 상황

    | 설계                | 결과                        |
    | ----------------- | ------------------------- |
    | PK만 있음            | 쓰기 빠름                     |
    | PK + UNIQUE 1개    | 약간 느림                     |
    | PK + UNIQUE 여러 개  | INSERT TPS 급락             |
    | PK 없음 + 여러 UNIQUE | 최악 (clustered index도 분산됨) |

7. 결론
- UNIQUE는 정말 필요한 것만: 내부 참조는 항상 PK(id), 외부에서 보이는 식별자만 UNIQUE
- UNIQUE는 읽기를 빠르게 해주지만, 쓰기를 비싸게 만든다: 한 row를 넣기 위해 여러 개의 B+Tree를 동시에 정렬해야 하기 때문

### [참고 문서] 엑셈 블로그 - DB 인사이드 | MySQL Architecture
- [1. MySQL 엔진](https://blog.ex-em.com/1679)
- [2. 스토리지 엔진](https://blog.ex-em.com/1680)
- [3. Thread](https://blog.ex-em.com/1681)
- [4. Memory](https://blog.ex-em.com/1682)
- [5. SQL 처리과정](https://blog.ex-em.com/1683)
- [6. InnoDB: In-Memory Structure](https://blog.ex-em.com/1698)
- [7. InnoDB: On-Disk Structure](https://blog.ex-em.com/1699)
- [8. InnoDB: 동작 원리](https://blog.ex-em.com/1700)
