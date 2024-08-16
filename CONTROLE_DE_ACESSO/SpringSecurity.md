# Registrar Componente na classe SecurityConfig

@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.csrf(csrf -> csrf.disable()); // Desativando segurança de ataque csrf. API Rest não guarda dados em sessão
    http.authorizeHttpRequests(auth -> auth.anyRequest().permitAll()); // Permitindo qualquer requisição.
    return http.build();
}

* Ao carregar esse Bean, estaremos permitindo acesso a todas requisições da aplicação, pois a ideia será registringir rotas específicas.

---------------------------------------------------------------------------------------------

# Implementando Interfaces do Spring Security nas classes User e Role

## Role

    public class Role implements <i>GrantedAuthority<i>

    Fazer Override do método getAuthority():

    @Override
    public String getAuthority() {
        return this.authority;
    }

## User

    public class User implements UserDetails

    Implementar todos os métodos da interface UserDetails (Os Override)

    * métodos boolean, fazer que todos retornem true, exemplo:

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    * método getUsername, retornar e-mail:

    @Override
    public String getUsername() {
        return email;
    }

    * método getAuthorities(), retornar roles:

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return roles;
    }



