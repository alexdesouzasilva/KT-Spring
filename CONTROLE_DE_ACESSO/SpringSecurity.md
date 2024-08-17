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

# Implementar Interface UserDetailsService ⭐


1. Criar classe UserService.
2. Implementar interface **UserDetailsService**
3. Implementar o método loadUserByUsername(String username)
4. Criar Projection para buscar os dados do banco de dados de forma eficiente.

    <span style="color: green"><strong>UserDetailsProjection</strong></span><br>
    String getUsername();<br>
	String getPassword();<br>
	Long getRoleId();<br>
	String getAuthority();<br>

5. Crirar interface **UserRepository** e adicionar Native Query:

    @Query(nativeQuery = true, value = """<br>
			SELECT tb_user.email AS username, tb_user.password, tb_role.id AS roleId, <br> tb_role.authority <br>
			FROM tb_user <br>
			INNER JOIN tb_user_role ON tb_user.id = tb_user_role.user_id <br>
			INNER JOIN tb_role ON tb_role.id = tb_user_role.role_id <br>
			WHERE tb_user.email = :email <br>
		""") <br>
    List<UserDetailsProjection> searchUserAndRolesByEmail(String email);

6. Agora no UserService, adicionar a lógica no método **loadUserByUsername**
    1. Buscar informações da base de dados
    List<UserDetailsProjection> result = repository.searchUserAndRolesByEmail(username);

    2. Verificar resultado, caso não seja o valor esperado, retornar exceção:
    if (result.size() == 0) {
        throw new UsernameNotFoundException("User not found");
    }

    3. Por fim, instanciar User e Role e retornar o user:
    User user = new User();
        user.setEmail(username);
        user.setPassword(result.get(0).getPassword());
        for (UserDetailsProjection projection : result) {
            user.addRole(new Role(projection.getRoleId(), projection.getAuthority()));
    }

    return user;

    <hr>
