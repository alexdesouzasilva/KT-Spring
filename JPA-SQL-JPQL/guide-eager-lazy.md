# EAGER e LAZY

## Carregamento padrão de entidades associadas
    * Para-um: EAGER ("Apressado") -> Traz consigo entidades relacionadas

    * Para-muitos: LAZY ("Preguiçoso") -> Não traz entidades relacionadas. Só será carregada se realmente precisar.


    Ao fazer uma requisição para buscar employees:
    http://localhost:8080/employees/1

    Que possui um relacionamento de muitos employees para um departamento:

    Está trazendo tanto os dados do funcionário quanto o de departamento do mesmo:

    Mesmo a busca sendo apenas para Employee:

    Method:
    @Transactional(readOnly = true)
	public EmployeeMinDTO findByIdMin(Long id) {
		Optional<Employee> result = repository.findById(id);
		return new EmployeeMinDTO(result.get());
	}

    Response:
    {
        "id": 1,
        "name": "Maria",
        "email": "maria@gmail.com",
        "department": {
            "id": 1,
            "name": "Sales"
        }
    }

    Query:

    SELECT
        emp.id,
        depart.id,
        depart.name,
        emp.email,
        emp.name 
    FROM
        tb_employee emp
    LEFT JOIN
        tb_department depart 
            ON depart.id=emp.department_id 
    WHERE
        emp.id=1

    Ou seja, por default, o relacionamento muitos employees para um departamente, está fazendo o left join de forma automática, diminuindo a performance da consulta.

    Response department:

        {
        "id": 1,
        "name": "Sales"
        }
    
    Consulta Department:

        SELECT
            depart.id,
            depart.name 
        FROM
            tb_department depart 
        WHERE
            depart.id=1

    Resumo:

        Quando faço uma consulta de employees MUITOS para UM department, o JPA faz a consulta EAGER, trazendo na consulta o department associado aos employees.

        No entanto, quando faço a consulta de UM department para MUITOS employes, o JPA faz a consulta LAZY, trazendo apenas os departments.

        O Comportamento LAZY faz a JPA enviar nova consulta ao banco de dados quando necessário, exemplo:

            1. Requisição para buscar todos funcionários do departamento de ID=1
            http://localhost:8080/departments/1/employees

            2. Method service responsável por realizar a busca:

            @Transactional(readOnly = true)
            public List<EmployeeMinDTO> findEmployeesByDepartment(Long id) {
                Optional<Department> result = repository.findById(id);
                List<Employee> list = result.get().getEmployees();
                return list.stream().map(x -> new EmployeeMinDTO(x)).toList();

            Ou seja, realiza a busca por department e depois faz o get em employees, logo, a JPA irá efetuar nova consulta no banco de dados para trazer todos funcionários do departamento. Por tanto, a busca será realizada se de fato for necessária.

## Alteração de comportamento

    * Não é indicado trocar os comportamentos default LAZY e EAGER.
    Contudo, a alteração seria na anotação de relacionamento:

    @OneToMany(mappedBy = "department", fech = FechType.EAGER)

## Ideal
    
    * Fazer consultas customizadas.

    Primeira Opção - Usando Join Fech no JPQL
    Query("SELECT obj FROM Employee obj JOIN FETCH obj.department")

