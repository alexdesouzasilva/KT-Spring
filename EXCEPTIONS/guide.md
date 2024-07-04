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
        public ResponseEntity<CustomError> resourceNotFound(ResourceNotFoundException e, HttpServletRequest request) { //Método passando a exceção e a captura da request
            HttpStatus status = HttpStatus.NOT_FOUND; // Status 404
            CustomError err = new CustomError(Instant.now(), status.value(), e.getMessage(), request.getRequestURI());//Instancia CustomError com os dados que serão informados na exceção.
            return ResponseEntity.status(status).body(err); // Retorna status mais o corpo do erro.
        }


        *** Exceção customizada para violação de integridade do Banco de dados ***

        @ExceptionHandler(DataBaseException.class)
        public ResponseEntity<CustomError> DataBaseException(DataBaseException e, HttpServletRequest request) {
            HttpStatus status = HttpStatus.BAD_REQUEST;
            CustomError err = new CustomError(Instant.now(), status.value(), e.getMessage(), request.getRequestURI());
            return ResponseEntity.status(status).body(err);
        }

## 5- Na classe service, fazer o tratamento da exceção nos métodos:

        *** ResourceNotFoundException ***
        try {
           (...)
        } catch (EntityNotFoundException e) {
            //Lançando minha exceção customizada
            throw new ResourceNotFoundException("Recurso não encontrado");
        }
    
        *** Por tanto, ao lançar EntityNotFoundException, iremos tratar capturando e lançando ResourceNotFoundException, nossa customização irá interceptar e lançar a exceção customizada.


        *** DataBaseException ***
         try {
            (...)
        } catch (DataIntegrityViolationException e) {
            throw new DataBaseException("Falha de integridade referencial");
        }

        *** Por tanto, ao capturar a exceção DataIntegrityViolationException, iremos tratar lançando DataBaseException, que na camada controller/handlers será devidamente tratada e lançada de forma customizado ao usuário.


# Customização para tratamento do Bean Validation

## 1 -  Nas entidades DTO's adicionar validação dos campos:
    *** Valida tamanho mínimo e máximo dos campos e obriga que não sejam nulos: ***

    @Size(min = 3, max = 80, message = "Nome precisa ter de 3 a 80 caracteres")
    @NotBlank(message = "Campo requerido") // Não pode ser null, não pode ser branco, ou seja, obrigatório.
    private String name;

    @Size(min = 10, message = "Descrição precisa ter no mínimo 10 caracteres")
    @NotBlank(message = "Campo requerido")
    private String description;

    *** Apenas valores positivos ***
    @Positive(message = "O preço deve ser positivo")
    private Double price;


## 2 - Criar classe DTO FieldMessage
    *** Classe responsável por tratar nome do campo e a mensagem validados ***

    private String fieldName;
    private String message;

## 3 - Criar classe DTO ValidationError
    a. ValidationError deverá herdar CustomError
    b. Deverá declarar uma lista de FieldMessage: List<FieldMessage>
    c. Construtor com os campos da classe Pai/Super
    d. Método get da lista
    e. Método para add objeto na lista

    public class ValidationError extends CustomError

        private List<FieldMessage> erros = new ArrayList<>();

        //Construtor obrigadado pela classe Pai/Super
        public ValidationError(Instant timestamp, Integer status, String error, String path) {
        super(timestamp, status, error, path);

        public void addError(String fieldName, String message) {
        erros.add(new FieldMessage(fieldName, message));
    
## 4 - Criar Método que interceptará exceções das validações Bean Validation

    //Sempre que MethodArgumentNotValidException for lançado, a seguinte tratativa será realizada
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<CustomError> MethodArgumentNotValidException(MethodArgumentNotValidException e, HttpServletRequest request) {
    HttpStatus status = HttpStatus.UNPROCESSABLE_ENTITY; // STATUS 422
    ValidationError err = new ValidationError(Instant.now(), status.value(), "Dados inválidos", request.getRequestURI());
    //Pegando campos e mensagens de erro capturado pela exceção, percorrendo cada campo e mensagem,
    //adicionando a lista de FieldError criada em DTO.
    for (FieldError f : e.getBindingResult().getFieldErrors()) {
        err.addError(f.getField(), f.getDefaultMessage()); 
    }
    return ResponseEntity.status(status).body(err); //Retornando status e corpo da mensagem contendo o erro, serializado em Json.
    }


