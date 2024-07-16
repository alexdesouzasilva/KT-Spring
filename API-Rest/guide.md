# Montar URI para retornar no location do Response.

1. Captura URI corrente: http://localhost:8080/clients
2. Adiciona o path /id retornado de dto: http://localhost:8080/clients/2
3. Retorna no header location do response

URI uri = ServletUriComponentsBuilder.fromCurrentRequest().path("/{id}")
.buildAndExpand(dto.getId()).toUri();

## Retorno usando ResponseEntity:
return ResponseEntity.create(uri).body(dto);