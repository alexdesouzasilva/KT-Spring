# Boas práticas para controlar sessões, usando JPA.

    Para que a sessão não fique aberta entre as camadas controller e service, usamos as configurações:

    No application properties:
    spring.jpa.open-in-view=false

    Os métodos da camanda service deverá ser anotado com:
    @Transactional

    Para métodos de consulta:
    @Transactional(readOnly = true)

