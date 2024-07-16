# EAGER e LAZY

## Carregamento padrão de entidades associadas
    * Para-um: EAGER ("Apressado") -> Traz consigo entidades relacionadas

    * Para-muitos: LAZY ("Preguiçoso") -> Não traz entidades relacionadas. Só será carregada se realmente precisar.


    Ao fazer uma requisição para buscar employees:
    http://localhost:8080/employees/1

    Que possui um relacionamento de muitos employees para um departamento:

    Está trazendo tanto os dados do funcionário quanto o de departamento do mesmo:

    {
    "id": 1,
    "name": "Maria",
    "email": "maria@gmail.com",
    "department": {
        "id": 1,
        "name": "Sales"
    }
}