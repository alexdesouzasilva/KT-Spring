#  Criando Exceções Customizadas 

## 1 - Identificar como o Framework retorna Exceções:

{
    "timestamp": "2024-06-29T08:32:00.369+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/products/26"
}

## 2 - Criar classe DTO para customizar as exceções CustomError
    a. Nela deverá conter os atributos mencionados acima:
    b. Os atributos deverão ser passados apenas por construtor, logo, a classe não terá setters.

    private Instant timestamp;
    private Integer status;
    private String error;
    private String path;


## 3 - Dentro do pacote service, criar pacote exceptions (Ou seja, exceções da camanda service)
### Exception para recurso não encontrado:
    ResourceNotFound exstends RuntimeException
    public ResourceNotFound(String msg) {
        super(msg)
    }


# !Importante
No pacote controller, criar um pacote handllers. Pacote que interceptará caso identifique alguma exceção.

## 4 - Criar classe ControllerException
    Obs.: O objetivo dessa classe é evitar diversos try-catch nas classe controllers.
    1. A classe deverá ser anotada com: ***@ControllerAdvice***

    2. Criar métodos customizado:
        *** Método para tratar erro 404 ***
        **** HttpServletRequest -> Usado para obter os dados da requisição ***

        @ExceptionHandler(ResourceNotFoundException.class)// Identifica tipo de erro lançado de service para controller
        public ResponseEntity<CustomError> resourceNotFound(ResourceNotFoundException e, HttpServletRequest request) {
            HttpStatus status = HttpStatus.NOT_FOUND; // Status 404
            CustomError err = new CustomError(Instant.now(), status.value(), e.getMessage(), request.getRequestURI());
            return ResponseEntity.status(status).body(err); // Retorna status mais o corpo do erro.
        }


        //Exceção customizada para violação de integridade do Banco de dados
        @ExceptionHandler(DataBaseException.class)
        public ResponseEntity<CustomError> DataBaseException(DataBaseException e, HttpServletRequest request) {
            HttpStatus status = HttpStatus.BAD_REQUEST;
            CustomError err = new CustomError(Instant.now(), status.value(), e.getMessage(), request.getRequestURI());
            return ResponseEntity.status(status).body(err);
        }

## 5- Na classe service, fazer o tratamente da exceção nos métodos:

        try {
           (...)
        } catch (EntityNotFoundException e) {
            //Lançando minha exceção customizada
            throw new ResourceNotFoundException("Recurso não encontrado");
        }
    
    *** Por tanto, ao lançar ResourceNotFoundException nossa customização irá interceptar e lança a exceção customizada.