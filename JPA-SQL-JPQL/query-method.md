Spring Data JPA:
https://spring.io/projects/spring-data-jpa

Query Methods:
https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html


JPQL:

SELECT obj
FROM Employee obj
WHERE UPPER(obj.name) LIKE 'MARIA%'

* obj -> precisa dar um apelido
* Employee -> Nome da classe
* Sempre acessar atributos pelo objeto.


PROJECTIONS:
Interfaces com campos que retornarão do banco de dados. Usado apenas quado estivermos usando Query Nativa.

public interface ProductMinProjection {
    String getName();
    Integer getAmount();
}

